---
type: concept
domain: architecture
tags: [aws, onion, networking, architecture]
---

# AWS-Onion — Mental Model

## The relationship-first model

```text
Internet
   ↓
Internet Gateway
   ↓
Route Table / Subnet
   ↓
NACL
   ↓
Security Group
   ↓
ENI / Private IP
   ↓
EC2
   ↓
Application / Service
```

> [!note]
> This is a conceptual dependency and troubleshooting model, not a literal sequence of independent physical boxes.

## What each layer answers

| Layer | Question |
|---|---|
| Internet | Where does traffic originate/go? |
| IGW | Is there an Internet doorway for the VPC? |
| Route Table | Is there a path to the destination network? |
| Subnet | Where does the resource live in the VPC CIDR? |
| NACL | Does subnet-level stateless filtering allow it? |
| Security Group | Does stateful resource-level filtering allow it? |
| ENI / IP | Which network interface/resource is addressed? |
| EC2 | Which virtual machine processes it? |
| Application | Which service is actually listening/responding? |

## Critical distinction

> **Route ≠ Address ≠ Permission**

A public subnet may have a route to an IGW while an EC2 instance has no public IPv4. Likewise, an address and route do not imply that Security Groups/NACLs permit traffic.

## Troubleshooting

```text
Connectivity?
   ↓
Route?
   ↓
Subnet controls?
   ↓
Security Group?
   ↓
Correct ENI/IP?
   ↓
Healthy EC2?
   ↓
Application listening?
```

## Related
- [[EC2 - Networking and Connectivity]]
- [[IAM - Groups Roles and STS]]
- [[Systems Manager - Mental Model]]
- [[CloudWatch - Mental Model]]
