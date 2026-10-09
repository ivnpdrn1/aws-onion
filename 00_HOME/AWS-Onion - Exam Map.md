---
type: map
project: AWS-Onion
tags: [aws, exam, architecture, developer, ai]
---

# AWS-Onion — Exam Map

## Solutions Architect

```text
VPC
 ↓
CIDR / Subnets
 ↓
Route Tables
 ↓
IGW / private connectivity
 ↓
NACL
 ↓
Security Groups
 ↓
ENI / IP
 ↓
EC2
 ↓
Application
```

### Three-pass exam method

```text
WHERE?
  ↓ Vertical layer
WHAT BUILDS IT?
  ↓ Horizontal component
WHAT HAPPENS NEXT?
  ↓ Functional effect
```

> **WHERE → WHAT INSIDE → WHAT NEXT**

### EC2 Nitro recognition

| Exam phrase | Association |
|---|---|
| underlying platform for modern EC2 | Nitro System |
| specialized hardware/modules | Nitro System |
| near bare-metal performance | Nitro |
| direct physical server access | Bare Metal |
| isolated sensitive compute | Nitro Enclaves |
| cryptographic attestation | Nitro Enclaves |
| no persistent storage / external networking | Nitro Enclaves |
| KMS + isolated processing | Nitro Enclaves |

### EC2 Pricing recognition

| Exam phrase | Association |
|---|---|
| unpredictable / no commitment | On-Demand |
| $/hour compute commitment | Savings Plans |
| broad EC2/Fargate/Lambda flexibility | Compute Savings Plan |
| EC2 family + Region commitment | EC2 Instance Savings Plan |
| configuration-based commitment | Reserved Instance |
| steady predictable workload | Savings Plans / RI |
| recurring fixed schedule | Scheduled Reserved — historical course concept |
| lowest cost + fault tolerant | Spot |
| two-minute interruption warning | Spot + EventBridge / metadata |
| target capacity using Spot + On-Demand | Spot Fleet / EC2 Fleet |
| guaranteed capacity in a specific AZ | Capacity Reservation / Zonal RI |
| physical isolation | Dedicated Instance |
| socket/core/host ID/BYOL | Dedicated Host |
| Spot Block | Historical/deprecated — do not choose for current design |

> **Price ≠ Capacity ≠ Tenancy ≠ Billing.**

> **Resource lifecycle ≠ Pricing commitment lifecycle.**

Related notes:
- [[AWS-Onion - Mental Model]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[EC2 - Nitro System and Nitro Enclaves]]
- [[EC2 - Pricing Options]]
- [[EC2 - Networking and Connectivity]]
- [[IAM - Groups Roles and STS]]
- [[Systems Manager - Mental Model]]
- [[CloudWatch - Mental Model]]

## Developer

The Developer layer will extend the Onion toward:

```text
Identity
 ↓
API / Event
 ↓
Application Logic
 ↓
State / Data
 ↓
Observability
 ↓
Deployment / Automation
```

Future subjects: Lambda, API Gateway, DynamoDB, SQS, SNS, EventBridge, Step Functions, SDK/CLI and CI/CD.

## AI

The AI layer will extend toward:

```text
Data
 ↓
Model Access
 ↓
Bedrock / AI Service
 ↓
Agent / Orchestration
 ↓
Tools
 ↓
Guardrails / IAM
 ↓
Observability / Cost
```

## Study rule

> If a service cannot yet be related **vertically to neighboring layers**, **horizontally to the components that build its specialty**, and **functionally to what happens when it acts**, the mental model is not finished.

> When using course material, preserve the lesson but also perform a **currency check** so historical AWS behavior is not mistaken for current operational guidance.
