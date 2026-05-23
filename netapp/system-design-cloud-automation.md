# System Design — Cloud Test Automation

---

## The 4-Step Framework

Use this structure for every system design question. Interviewers evaluate **how you think, communicate tradeoffs, and handle ambiguity** — not whether your answer is perfect.

```
Step 1 → Outline use cases, constraints, assumptions   (never skip this)
Step 2 → High-level design
Step 3 → Design core components
Step 4 → Scale the design
```

Never start drawing boxes until you've completed Step 1. An interviewer who asks "design a cloud test automation system" has left 80% deliberately underspecified — your job is to define it before solving it.

---

## Step 1 — Use Cases, Constraints, Assumptions

Ask questions to scope the problem. State your assumptions out loud so both sides are working on the same problem.

### Questions to ask (NetApp cloud automation context)

**Who uses it and how?**
- Triggered by developers pushing code, a nightly schedule, or both?
- Who consumes results — engineers, QA team, dashboards?
- Multiple teams sharing the same system or per-team?

**What scale?**
- How many test runs per day? 10? 1000?
- How long is a typical run? 10 minutes? 2 hours?
- How many tests per run? How many parallel workers needed?
- How many cloud resources provisioned per run (1 EC2? 20 EC2s)?

**What clouds?**
- AWS only, or multi-cloud (AWS + GCP + Azure)?
- Tests on all three simultaneously or one at a time?

**Reliability and speed requirements?**
- If provisioning fails, do we retry? How many times?
- Acceptable time from code push to results available?
- Real-time status or is polling acceptable?

**What does "done" look like?**
- Results stored where — S3? Database? Dashboard?
- Historical trend data needed (pass rates over time)?
- Who gets notified on failure — Slack, email, both?

### State your assumptions

> "I'll assume ~100 test runs/day across AWS and GCP, each run lasts ~30 minutes, results go to S3 with Slack notification, 24/7 availability with automated retries on failure."

---

## Step 2 — High-Level Design

Draw the main components and data flow first. Don't detail anything yet.

```
Developer push / Scheduler
        │
        ▼
  CI Trigger (Jenkins / GitHub Actions)
        │
        ▼
  Test Orchestrator
  ┌─────┴──────┐
  │            │
  ▼            ▼
Provision   Queue jobs
(Terraform   (SQS/
 / boto3)     Pub/Sub)
  │            │
  ▼            ▼
Cloud       Test Workers (EC2 / K8s Jobs)
Resources       │
                ▼
          Collect Results
                │
        ┌───────┴────────┐
        ▼                ▼
     Store (S3)       Notify (Slack/SNS)
        │
        ▼
   Dashboard / DB
   (DynamoDB / RDS)
```

**Key flows:**
1. CI system detects push or schedule fires
2. Orchestrator calls Terraform/boto3 to provision cloud resources
3. Test workers execute tests against provisioned infra
4. Results collected, stored in S3 as JUnit XML + summary JSON
5. Notification sent; infra torn down regardless of result

---

## Step 3 — Core Components

### CI Trigger Layer
Jenkins or GitHub Actions. Handles the event and kicks off the pipeline. Passes parameters (branch, cloud target, test suite) to the orchestrator.

### Test Orchestrator
The brain. Responsibilities:
- Calls provisioning layer with required resource spec
- Submits test jobs to workers
- Tracks run state (pending → provisioning → running → collecting → done)
- Calls teardown after run regardless of outcome
- Retries on transient failures with backoff

State stored in **DynamoDB** (run ID → status, timestamps, result location).

### Provisioning Layer
Terraform for multi-cloud (declarative, consistent). boto3/SDK for dynamic per-run resources (e.g. spin up exactly N EC2s based on test count). Use **Terraform workspaces** or separate state per run to avoid conflicts between parallel runs.

### Test Workers
EC2 instances or **Kubernetes Jobs** (preferred — K8s handles scheduling, retries, cleanup automatically). Each worker:
- Pulls test container from ECR/GCR
- Runs assigned test subset
- Writes JUnit XML output to S3

### Result Collection
- S3 bucket per run: `s3://results/<run-id>/`
- JUnit XML for CI integration (Jenkins `junit` step, GitHub Actions test summary)
- Summary JSON with pass/fail counts, duration, resource IDs used
- DynamoDB record updated with result location

### Notification
SNS topic → Lambda → Slack webhook. Lambda formats the message with run summary, pass rate, S3 link. For email: SNS direct subscription.

### Teardown
Always in `post { always }` (Jenkins) or equivalent. Even if tests crash halfway, Terraform destroys what was provisioned. Track provisioned resource IDs in DynamoDB so teardown can target exactly what was created by this run.

---

## Step 4 — Scale the Design

### Identify bottlenecks first

| Bottleneck | Problem |
|---|---|
| Provisioning time | Terraform `apply` takes 3–5 min for EC2 → slow feedback |
| Parallel capacity | 100 runs × 20 EC2s = 2000 instances peak |
| Result aggregation | 10,000 test result files per run → slow reads |
| State contention | Concurrent runs modifying shared Terraform state |

### Solutions

**Reduce provisioning latency:**
- Pre-warm a pool of EC2 instances in a "ready" state; assign to runs instead of cold-provisioning
- Use Spot instances for test workers (80% cheaper; use interruption handling to requeue)
- Containerize workers on K8s — pod startup is seconds vs EC2 minutes

**Parallel capacity:**
- Auto Scaling Groups with target tracking — cluster scales with queue depth
- Use K8s cluster autoscaler on EKS; nodes added as job queue grows

**State contention:**
- Separate Terraform state per run (S3 key: `tfstate/<run-id>/terraform.tfstate`)
- DynamoDB table locking per run — concurrent runs never share state

**Result aggregation at scale:**
- Partition S3 by `run-id/worker-id/` for parallel writes
- Athena for ad-hoc queries over S3 result files without loading into DB
- DynamoDB stores metadata + pointers; S3 stores raw results

**Reliability:**
- Dead letter queue (SQS DLQ) for failed jobs — inspect without losing them
- Idempotent job IDs — retrying a job that half-ran doesn't double-count results
- Circuit breaker on cloud provisioning — if AWS API is throwing errors, stop queueing new runs rather than piling up failures

**Multi-region:**
- Route runs to the cloud region closest to the test target
- S3 cross-region replication for result durability
- Single DynamoDB global table for run state across regions

### Numbers to size the system (example assumptions)

```
100 runs/day
× 30 min avg duration
× 20 EC2s per run
= peak ~2000 instances if all concurrent (unlikely)
= ~60 runs in flight at steady state (100/day ÷ 24h × 30min overlap)
= ~1200 EC2s steady state peak
→ Use Auto Scaling with max 1500, Spot with On-Demand fallback
```

---

## Quick Reference — Design Tradeoffs to Know

| Decision | Option A | Option B | Pick when |
|---|---|---|---|
| Provisioning | Terraform | boto3 | Terraform for consistency/multi-cloud; boto3 for dynamic per-run tweaks |
| Workers | K8s Jobs | EC2 | K8s if cluster exists; EC2 for heavy VMs or GPU tests |
| Queue | SQS | Kafka | SQS for simple task queue; Kafka if you need replay/stream processing |
| Results DB | DynamoDB | RDS | DynamoDB for run metadata (key-value, auto-scale); RDS if you need complex queries/joins |
| Notification | SNS→Lambda | Direct webhook | SNS for fan-out to multiple subscribers; direct webhook if single destination |
