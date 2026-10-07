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

Related notes:
- [[AWS-Onion - Mental Model]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[EC2 - Nitro System and Nitro Enclaves]]
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
