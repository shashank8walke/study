# AWS vs Azure vs GCP

---

## Service Mapping

| Category | AWS | Azure | GCP |
|---|---|---|---|
| **Compute** | EC2 | Virtual Machines | Compute Engine (GCE) |
| **Managed K8s** | EKS | AKS | GKE |
| **Serverless** | Lambda | Azure Functions | Cloud Functions / Cloud Run |
| **Object Storage** | S3 | Blob Storage | Cloud Storage (GCS) |
| **Block Storage** | EBS | Managed Disks | Persistent Disk |
| **Container Registry** | ECR | ACR | Artifact Registry |
| **VPC/Networking** | VPC | Virtual Network (VNet) | VPC |
| **Load Balancer** | ALB / NLB | Application Gateway / LB | Cloud Load Balancing |
| **IAM** | IAM + STS | Entra ID (AAD) + RBAC | Cloud IAM |
| **Relational DB** | RDS / Aurora | Azure SQL Database | Cloud SQL |
| **NoSQL DB** | DynamoDB | Cosmos DB | Firestore / Bigtable |
| **Data Warehouse** | Redshift | Synapse Analytics | BigQuery |
| **Message Queue** | SQS | Service Bus | Pub/Sub |
| **Event Streaming** | Kinesis | Event Hubs | Pub/Sub + Dataflow |
| **DNS** | Route 53 | Azure DNS | Cloud DNS |
| **CDN** | CloudFront | Azure CDN / Front Door | Cloud CDN |
| **Secret Management** | Secrets Manager | Key Vault | Secret Manager |
| **Monitoring/Logging** | CloudWatch | Azure Monitor | Cloud Monitoring / Logging |
| **CI/CD** | CodePipeline + CodeBuild | Azure DevOps Pipelines | Cloud Build |
| **IaC (native)** | CloudFormation | ARM Templates / Bicep | Deployment Manager |
| **IaC (cross-cloud)** | Terraform | Terraform | Terraform |
| **ML Platform** | SageMaker | Azure ML | Vertex AI |
| **CLI** | `aws` | `az` | `gcloud` |

---

## AWS

### Strengths
- **Widest service catalog** — 200+ services, most mature ecosystem
- **Largest market share** (~32%) — most enterprises, most job demand
- **Most third-party integrations** — every tool supports AWS first
- **Best for: general cloud workloads, enterprise migrations, startups**

### Key concepts not in notes yet

**Regions and AZs:** Each region has 2–6 Availability Zones. AZs are isolated data centers within a region. Deploy across multiple AZs for HA.

**CloudWatch:** Metrics, logs, alarms, dashboards. `boto3` can put custom metrics:
```python
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[{'MetricName': 'TestsPassed', 'Value': 95, 'Unit': 'Count'}]
)
```

**ELB types:**
- ALB (Application LB) — Layer 7, HTTP/HTTPS, path-based routing
- NLB (Network LB) — Layer 4, TCP/UDP, ultra-low latency, static IP

**SQS vs SNS:**
- SQS: queue (pull model) — workers poll for messages, one consumer per message
- SNS: pub/sub (push model) — fan-out to multiple subscribers (SQS, Lambda, email, HTTP)
- Pattern: SNS → multiple SQS queues (fan-out + decoupling)

---

## Azure

### Strengths
- **Microsoft ecosystem integration** — Active Directory, Office 365, .NET, Windows Server native
- **Hybrid cloud** — Azure Arc manages on-prem and multi-cloud from Azure. Best hybrid story.
- **Enterprise AD/identity** — Entra ID (formerly AAD) is the gold standard for enterprise SSO/MFA
- **Best for: Microsoft shops, enterprises with existing AD, hybrid cloud needs**

### Key concepts

**Entra ID (Azure Active Directory):**
Azure's identity platform. Unlike AWS IAM (which is only for cloud resources), Entra ID also handles: employee logins, SSO for SaaS apps, MFA, conditional access.

In AWS you'd use IAM for service permissions + Cognito or a third-party IdP for user auth. In Azure, Entra ID covers both.

**RBAC:**
Role-Based Access Control. Assign built-in or custom roles to users/service principals at scope levels: Management Group → Subscription → Resource Group → Resource.

```
AWS IAM Policy ≈ Azure Role Definition
AWS IAM Role   ≈ Azure Service Principal
AWS Account    ≈ Azure Subscription
AWS OU         ≈ Azure Management Group
```

**Resource Groups:**
In Azure, every resource must live inside a Resource Group — a logical container for related resources. No equivalent in AWS (closest is tags + IAM boundaries). Resource Groups enable bulk operations: delete the whole group to tear down an environment.

**VNet (Virtual Network):**
Azure's VPC equivalent. Subnets live inside a VNet. Unlike AWS VPCs (regional, isolated by default), Azure VNets can peer across regions. Network Security Groups (NSGs) are like AWS Security Groups.

**Azure Arc:**
Extends Azure management (policy, RBAC, monitoring) to on-prem servers, VMs on other clouds, or K8s clusters anywhere. Lets you manage a multi-cloud environment from a single Azure control plane.

**Blob Storage:**
Equivalent to S3. Three tiers: Hot (frequent access), Cool (infrequent), Archive (like Glacier). Access via storage account → container → blob.

**AKS (Azure Kubernetes Service):**
Managed K8s. Control plane is free (you pay only for worker nodes). Integrates with Entra ID for K8s RBAC, Key Vault for secrets.

**Useful CLI:**
```bash
az login
az account set --subscription "my-sub"
az group create --name myRG --location eastus
az vm list --resource-group myRG --output table
az aks get-credentials --resource-group myRG --name myCluster
```

---

## GCP

### Strengths
- **Best managed Kubernetes** — GKE is the most mature (Google invented K8s)
- **Best data + ML** — BigQuery (serverless data warehouse), Vertex AI, Dataflow
- **Global VPC** — single VPC spans all regions without peering (unique to GCP)
- **Networking** — Google's private fiber backbone; lowest latency for global traffic
- **Best for: data engineering, ML workloads, companies already using Google Workspace, K8s-first teams**

### Key concepts

**Global VPC:**
In AWS/Azure, a VPC/VNet is regional — you create separate ones per region and peer them. In GCP, a single VPC is global by default. A subnet in `us-east1` and a subnet in `europe-west1` can be in the same VPC with private connectivity — no peering setup needed.

**IAM:**
GCP IAM uses email-based identities:
- **Service Account** (machine identity) ≈ AWS IAM Role
- **Member** can be a Google account, group, service account, or domain
- Roles at resource, project, folder, or org level

```bash
gcloud auth activate-service-account --key-file=key.json
gcloud config set project my-project
```

**GKE (Google Kubernetes Engine):**
The most battle-tested managed K8s. Autopilot mode: fully managed, you only define pods, GCP manages nodes. Standard mode: you manage node pools. Supports Workload Identity (pods authenticate to GCP APIs without service account keys).

**BigQuery:**
Serverless, columnar data warehouse. No infrastructure to manage — just run SQL queries against petabyte-scale data. Pay per query (bytes scanned). Unique to GCP; closest AWS equivalent is Redshift (but BQ is serverless, Redshift is a cluster you provision).

```sql
SELECT test_name, COUNT(*) as runs, AVG(duration_sec) as avg_duration
FROM `my-project.test_results.runs`
WHERE DATE(timestamp) = CURRENT_DATE()
GROUP BY test_name
```

**Cloud Run:**
Fully managed serverless containers. You give it a container image; it runs it on HTTP request, scales to zero. No K8s to manage. Between Lambda (function-level) and K8s (full orchestration). Great for test result processors, webhook handlers.

**Pub/Sub:**
GCP's messaging service. Acts as both SQS (queue) and SNS (pub/sub). Guaranteed at-least-once delivery. Used heavily in data pipelines.

**Useful CLI:**
```bash
gcloud compute instances list
gcloud container clusters get-credentials my-cluster --region us-central1
gsutil ls gs://my-bucket/           # Cloud Storage CLI
gsutil cp file.txt gs://my-bucket/  # upload
bq query --use_legacy_sql=false 'SELECT ...'  # BigQuery CLI
```

---

## What's Common Across All Three

- **VPC-equivalent networking** — isolated private networks, subnets, firewall rules
- **Object storage** — S3 / Blob / GCS: same model (key-value, metadata, lifecycle policies, presigned/SAS URLs)
- **Managed Kubernetes** — EKS / AKS / GKE: all run standard K8s, differ in maturity and integration
- **Serverless compute** — Lambda / Functions / Cloud Functions+Cloud Run
- **IAM/RBAC** — all use roles + policies, all support temporary credentials for machine identities
- **Managed databases** — relational + NoSQL options on all three
- **Container registries** — ECR / ACR / Artifact Registry: push images, pull in K8s
- **Terraform support** — works across all three with provider plugins
- **CLI tools** — `aws`, `az`, `gcloud`: similar patterns (list, create, describe, delete)
- **Pay-as-you-go** — all bill by usage (compute hours, storage GB, requests)
- **Global regions and availability zones** — all have multi-region, multi-AZ deployments
- **Monitoring + Logging** — CloudWatch / Azure Monitor / Cloud Logging: metrics, logs, alerts

---

## Key Differences

| Aspect | AWS | Azure | GCP |
|---|---|---|---|
| **Identity** | IAM separate from user auth | Entra ID handles both infra + user auth | IAM via service accounts |
| **VPC scope** | Regional (per region) | Regional (VNet) | Global (one VPC spans all regions) |
| **Resource grouping** | Tags + accounts | Resource Groups (mandatory) | Projects |
| **K8s maturity** | EKS (good) | AKS (good) | GKE (best — invented K8s) |
| **Data warehouse** | Redshift (provisioned cluster) | Synapse (provisioned) | BigQuery (serverless, pay-per-query) |
| **Hybrid cloud** | Outposts (limited) | Azure Arc (best) | Anthos |
| **Market share** | ~32% (largest) | ~23% (2nd) | ~12% (3rd, fastest growing) |
| **Pricing** | Complex, many options | Complex, Microsoft discounts | Often cheaper for compute, sustained-use discounts automatic |

---

## When to Use What

**Choose AWS when:**
- General-purpose cloud workloads
- Widest service selection needed (niche ML, IoT, media, etc.)
- Largest hiring pool and third-party ecosystem
- No strong preference for another vendor

**Choose Azure when:**
- Already invested in Microsoft ecosystem (Office 365, Active Directory, .NET, Windows Server)
- Strong enterprise identity/compliance requirements
- Hybrid cloud: extending on-prem to cloud
- Microsoft SQL Server or .NET workloads

**Choose GCP when:**
- Data engineering and ML are the primary workload (BigQuery, Vertex AI, Dataflow)
- Kubernetes is the core platform (GKE Autopilot is the easiest managed K8s)
- Global networking performance is critical
- Already using Google Workspace

**Multi-cloud (relevant for NetApp):**
- Use Terraform for consistent provisioning across clouds
- Abstract cloud-specific details behind a common interface in your automation code
- Common pattern: primary workloads on AWS, analytics on GCP (BigQuery), enterprise identity on Azure AD
