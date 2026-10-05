# AWS-Onion — Downstream Model (Aguas Abajo)

## Principle
AWS-Onion follows **causality and dependency from the origin of a flow toward its result**.

> **#1 is always the origin/initiator of the specific flow being modeled.**

Numbering follows the logical sequence: **who starts → which service receives → how it travels → which controls apply → where it executes → under which authorization → how the result is observed.**

## Canonical questions
```text
#1 WHO STARTS?
 ↓
#2 WHO COORDINATES / RECEIVES?
 ↓
#3 THROUGH WHICH SERVICE ACCESS PATH?
 ↓
#4 CAN IT REACH THE DESTINATION?
 ↓
#5 WHERE / BY WHOM IS IT EXECUTED?
 ↓
#6 IS THAT ACTION AUTHORIZED?
 ↓
#7 HOW IS THE RESULT OBSERVED / AUDITED?
```

IAM and observability can be **transversal layers**. Their number explains causal flow; it does not mean IAM is physically after compute.

## Example — EC2 administration with Session Manager

| # | AWS-Onion layer | Component | Core question |
|---:|---|---|---|
| 1 | User / Identity | Administrator (IAM User / Role) | Who starts? |
| 2 | Management / Service | Systems Manager → Session Manager | Who coordinates? |
| 3 | Service Endpoint | Interface VPC Endpoints: ssm, ssmmessages, ec2messages where applicable | Through which private AWS service path? |
| 4 | Network | VPC → Private Subnet → endpoint/path → NACL → SG → ENI/Private IP | Can it reach? |
| 5 | Compute / Execution | SSM Agent → EC2 | Who executes? |
| 6 | Identity & Permission | IAM Instance Role + policies | Is the action authorized? |
| 7 | Observability | CloudWatch Logs | What happened? |

### #1 User / Identity
The administrator starts a Session Manager session from AWS Console or CLI. Direct network access to EC2 is not required.

### #2 Management / Service
Systems Manager / Session Manager receives and coordinates the management session.

### #3 Service Endpoint
In the private-subnet lab design, Interface VPC Endpoints provide private connectivity to required Systems Manager services. The lab model uses HTTPS/TCP 443.

### #4 Network
```text
VPC
 ↓
Private Subnet
 ↓
VPC Endpoint / routing path
 ↓
NACL (if applicable)
 ↓
Security Group
 ↓
ENI / Private IP
```
For the interface endpoints in this practice, the endpoint Security Group permits HTTPS/TCP 443 from the appropriate VPC source/CIDR.

> **Route ≠ Address ≠ Permission.**

### #5 Compute / Execution
SSM Agent runs on EC2 and participates in the Systems Manager session. Session Manager therefore does not require inbound SSH/TCP 22 to the instance.

### #6 Identity & Permission
The IAM Instance Role supplies AWS permissions required by EC2/SSM Agent when interacting with AWS services.

```text
EC2 / SSM Agent
       ↕
IAM Instance Role
       ↓
AWS APIs
```

The Role is a **transversal authorization layer**, not a physical network hop.

### #7 Observability
Session logging can send activity to Amazon CloudWatch Logs, for example a log group such as `session-manager-logs`.

CloudWatch is a **transversal observability layer**, not another packet-routing hop.

## Complete downstream view
```text
#1 ADMINISTRATOR
   WHO STARTS?
       │
       ▼
#2 SYSTEMS MANAGER / SESSION MANAGER
   WHO COORDINATES?
       │
       ▼
#3 VPC ENDPOINTS
   THROUGH WHICH PRIVATE SERVICE PATH?
       │
       ▼
#4 VPC / SUBNET / SG / NACL / ENI / PRIVATE IP
   CAN IT REACH?
       │
       ▼
#5 SSM AGENT + EC2
   WHO EXECUTES?
       │
       │◄──── #6 IAM INSTANCE ROLE
       │      IS IT AUTHORIZED?
       ▼
#7 CLOUDWATCH LOGS
   WHAT HAPPENED?
```

## AWS-Onion mental formula
> **IDENTITY → SERVICE → ACCESS PATH → CONNECTIVITY → EXECUTION → AUTHORIZATION → OBSERVABILITY**

> **WHO? → WHICH SERVICE? → HOW DOES IT GET THERE? → CAN IT REACH? → WHO EXECUTES? → CAN IT DO IT? → WHAT HAPPENED?**

This downstream convention is the default numbering model for future AWS-Onion diagrams.
