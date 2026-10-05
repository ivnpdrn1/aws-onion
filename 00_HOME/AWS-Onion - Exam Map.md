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

Related notes:
- [[AWS-Onion - Mental Model]]
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

> If a service cannot yet be related to the layers around it, the mental model is not finished.
