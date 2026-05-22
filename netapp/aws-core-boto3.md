# AWS Core Services + boto3

---

## EC2 — Elastic Compute Cloud

Virtual server (e.g. `t2.micro`). Lives in a specific **Availability Zone**.

| Concept | Detail |
|---|---|
| **AMI** | OS template (Amazon Machine Image) used to launch an instance |
| **EBS** | Elastic Block Storage — persistent disk, survives stop. "Delete on terminate" is optional. |
| **Instance Store** | Temp local disk — wiped on stop/terminate. Fast but ephemeral. |

### Auto Scaling Groups (ASG)

Automatically launches/terminates EC2s based on load.

**Health checks:**
- **Instance check** — detects hardware/OS failure
- **ELB check** — detects if the app/endpoint is actually responding

**Scaling policies:**

| Type | Trigger |
|---|---|
| Target tracking | Keep CPU at X% |
| Step scaling | Scale by N when threshold crossed |
| Scheduled | Time-based |
| Predictive | ML forecasts load |

**Lifecycle Hooks** — intercept launch or termination to run custom actions before the instance goes live or is removed:
- "Before this new instance goes live, register it with our test framework"
- "Before this instance terminates, pull its test logs to S3"

Highly relevant for automation pipelines — lets you hook infra events into your CI/CD workflow.

### Placement Groups

| Type | Use case |
|---|---|
| Cluster | Low latency, same rack — HPC |
| Spread | Instances on different hardware — HA |
| Partition | Groups of instances on separate racks — large distributed systems |

### Instance Metadata

EC2 can query its own metadata at runtime instead of hardcoding values:
```bash
curl http://169.254.169.254/latest/meta-data/instance-id
curl http://169.254.169.254/latest/meta-data/public-ipv4
```
Useful in automation scripts running on the instance itself.

---

## S3 — Simple Storage Service

Object storage. Accessed via URLs/APIs. **Not inside a VPC** — it's a global service accessed via internet or VPC endpoints.

**Object** = `key` (path) + `value` (data) + `metadata`

> There are no real folders in S3. The `/` in keys just makes the console display them folder-like. It's all flat keys under the hood.

### Storage Classes

| Class | Use case |
|---|---|
| Standard | Frequently accessed |
| Standard-IA | Infrequent access, lower cost |
| One Zone-IA | IA but single AZ only |
| Glacier / Glacier Deep Archive | Archival, retrieval takes minutes to hours |

### Access Control

- **Private by default**
- **Bucket policies** — JSON resource-based policies
- **Presigned URLs** — temporary URL for a private object; no AWS creds needed by recipient
- **Block Public Access** — account/bucket-level override that blocks all public access regardless of policies

### Lifecycle Policies

Automate transitions between storage classes or deletion based on object age.

---

## IAM — Identity and Access Management

Follow **least privilege** — grant only what's needed.

| Entity | For |
|---|---|
| **Users** | Humans / client machines with long-lived credentials |
| **Roles** | AWS services, EC2 instances, Lambda functions |
| **Policies** | JSON documents defining allow/deny on resources |

Policy types: AWS managed → Customer managed → Inline (attached directly to a single entity).

> **Never hardcode Access Keys in source code.** Use IAM Roles instead — boto3 picks up role credentials automatically on EC2.

---

## STS — Security Token Service

IAM Roles work via STS under the hood. When an EC2 **assumes a Role**, STS issues **temporary credentials** (Access Key + Secret + Session Token) that expire after a configurable duration (15 min to 12 hours).

This is far more secure than long-lived static keys — credentials rotate automatically and are scoped to the role's permissions.

---

## VPC — Virtual Private Cloud

Your private isolated network in AWS.

```
Internet
    │
Internet Gateway
    │
Public Subnet  ←── EC2, Load Balancers (have public IPs, route to IGW)
    │
NAT Gateway    ←── lets private subnet reach internet (outbound only)
    │
Private Subnet ←── Databases, internal services (no inbound from internet)
```

**Security Groups** — stateful firewall rules attached to EC2 instances.
- Stateful = if you allow inbound, the return traffic is automatically allowed.
- Example: allow TCP 22 (SSH), TCP 443 (HTTPS)

**NAT Gateway** — private subnet resources can initiate outbound internet connections (e.g. download packages) without being reachable from the outside.

---

## boto3 — AWS SDK for Python

```python
import boto3
```

### Authentication

- **Local:** uses `~/.aws/credentials` or env vars
- **On EC2 with IAM Role:** boto3 automatically fetches temporary credentials from the instance metadata service — no config needed, preferred approach

### Client vs Resource

| | Client | Resource |
|---|---|---|
| Level | Low-level, mirrors AWS API exactly | High-level, Pythonic objects |
| Returns | Python dicts | Objects with attributes + methods |
| Coverage | All operations | Not all operations |
| Use when | Need raw response or unsupported op | Readability in most code |

```python
# Client
ec2_client = boto3.client('ec2', region_name='us-east-1')
response = ec2_client.describe_instances()           # returns dict
instances = response['Reservations'][0]['Instances']

# Resource
ec2 = boto3.resource('ec2', region_name='us-east-1')
for instance in ec2.instances.all():
    print(instance.id, instance.state)              # objects with attributes
```

### Sessions

A Session stores credentials, region, and profile. Essential for multi-region automation:

```python
session_us = boto3.Session(region_name='us-east-1')
session_eu = boto3.Session(region_name='eu-west-1')
ec2_us = session_us.client('ec2')
ec2_eu = session_eu.client('ec2')
```

### EC2 Operations

```python
ec2 = boto3.resource('ec2')

# Launch
instance = ec2.create_instances(
    ImageId='ami-0abcdef1234567890',
    InstanceType='t2.micro',
    MinCount=1, MaxCount=1
)[0]

# Start / Stop
instance.start()
instance.stop()

# Filter running instances
running = ec2.instances.filter(
    Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
)
```

**Waiters** — poll AWS until a resource reaches the desired state. Never write manual polling loops:

```python
client = boto3.client('ec2')
waiter = client.get_waiter('instance_running')
waiter.wait(InstanceIds=[instance_id])   # blocks until instance is running
# continues here only once state is confirmed
```

### S3 Operations

```python
s3 = boto3.client('s3')

s3.create_bucket(Bucket='my-bucket')
s3.upload_file('local.txt', 'my-bucket', 'remote/key.txt')
s3.download_file('my-bucket', 'remote/key.txt', 'local.txt')

for obj in s3.list_objects_v2(Bucket='my-bucket')['Contents']:
    print(obj['Key'])

s3.delete_object(Bucket='my-bucket', Key='remote/key.txt')
s3.copy_object(CopySource={'Bucket': 'src', 'Key': 'k'}, Bucket='dst', Key='k')
```

**Presigned URLs** — temporary URL for a private object, no AWS credentials needed by the recipient:

```python
url = s3.generate_presigned_url(
    'get_object',
    Params={'Bucket': 'my-bucket', 'Key': 'report.pdf'},
    ExpiresIn=3600   # seconds
)
# share this URL with anyone — expires after 1 hour
```

### Paginators

AWS APIs return paginated results (limited per response). Always use paginators when listing — missing pagination = **silent data loss**:

```python
paginator = s3.get_paginator('list_objects_v2')
for page in paginator.paginate(Bucket='my-bucket'):
    for obj in page.get('Contents', []):
        print(obj['Key'])
```

### Error Handling

```python
import botocore

try:
    s3.get_object(Bucket='my-bucket', Key='missing.txt')
except botocore.exceptions.ClientError as e:
    code = e.response['Error']['Code']
    if code == 'NoSuchKey':
        print("Object not found")
    elif code == '403':
        print("Access denied")
    else:
        raise
```

### Key Interview Takeaways

- **Client vs Resource** — the single most asked boto3 question. Client = low-level dicts, Resource = high-level objects.
- **Waiters** — never write manual polling loops when a waiter exists.
- **Paginators** — always use when listing. Missing pagination = silent data loss.
- **IAM Roles over access keys** — on EC2, boto3 picks up role credentials automatically.
- **Session for multi-region** — use explicit sessions when automation spans regions.
- **Error handling** — always catch `ClientError` and inspect `e.response['Error']['Code']`.
