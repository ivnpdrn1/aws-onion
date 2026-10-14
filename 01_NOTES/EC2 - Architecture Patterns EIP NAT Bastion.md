---
type: architecture-pattern
domain: ec2-networking
updated: 2026-10-14
tags: [aws-onion, ec2, elastic-ip, nat-gateway, bastion, session-manager, high-availability, exam]
---
# EC2 Architecture Patterns — EIP, NAT Redundancy and Bastion

> **Three different needs:** stable public identity ≠ redundant private egress ≠ privileged administration.

## Requirement → decision → why not something else

| Requirement | Classroom solution | Why this works | Competing answers |
|---|---|---|---|
| Single EC2 uses static public IP; move address to failover EC2 on failure | **Elastic IP (EIP)** | Regional static IPv4 can be reassociated to compatible replacement instance/ENI | Auto-assigned IPv4 changes after stop/start; ALB has no directly assigned fixed EIP; Route 53 changes DNS not necessarily same address |
| Private EC2 in several AZ require resilient outbound IPv4 internet path | **Public zonal NAT Gateway per AZ + AZ-specific routes** | Each private subnet route points to an AZ-local NAT; one AZ failure need not break another | Single zonal NAT introduces AZ dependency; IGW alone doesn't give private-IPv4-only EC2 internet egress; NAT Instance needs maintenance |
| Engineers need remote SSH to private EC2 | **Bastion host in public subnet** | Controlled public SSH entry; Bastion connects to private EC2 over local VPC network | NAT Gateway is outbound translation not inbound SSH; ALB is L7 HTTP(S); modern alternatives EIC Endpoint or SSM Session Manager |

## A: EIP failover — functional sequence

```text
External client → stable EIP → EC2-A
   failure detected → automation/operator reassociates EIP to compatible EC2-B
External client → same EIP → EC2-B
```

**Why:** public IPv4 address stays stable even if workload compute is replaced. **Important:** EIP reassociation **is not automatic failure detection**; requires health signals plus failover automation or operator. It stays in one AWS Region and AWS bills for public IPv4. EIP only preserves addressing, not storage or application state. For application-level HA, ALB + target groups / Route 53 / Global Accelerator may be better depending on needs.

## B: NAT HA — vertical, horizontal and functional onion

```text
AZ-A: private EC2 → subnet route 0.0.0.0/0 → NAT-A (public subnet) → IGW → internet
AZ-B: private EC2 → subnet route 0.0.0.0/0 → NAT-B (public subnet) → IGW → internet
```

### Why every link?
| Link | Function | Implementation |
|---|---|---|
| ENI / private IP | Address of EC2 | Attach ENI in private subnet |
| Private subnet Route Table | Direct default route to appropriate NAT | 0.0.0.0/0 → nat-ID of same AZ |
| Public NAT Gateway zonal | NAT of private-originated traffic | Create in public subnet and associate EIP |
| Public subnet Route Table | External default route | 0.0.0.0/0 → igw-ID |
| IGW | VPC connectivity to Internet and public IPv4 mapping | Attach to VPC |
| SG/NACL | Permit required connections | Configure source/destination protocols and return traffic |

If all private subnets across AZs route to NAT-A, failure of AZ-A can break outbound access from surviving AZ-B. **Class answer assumes zonal NAT Gateway.** A **Regional NAT Gateway** is a newer AWS alternative worth evaluating for region-level automatic multi-AZ scalability. AWS bills NAT gateways; consider VPC Endpoints for eligible AWS services.

## C: Bastion SSH — functional sequence

```text
Engineer (authorized IP)
  → Internet → IGW → public subnet Bastion EC2 (EIP or public IP, SG allows SSH 22)
  → local VPC route → private EC2 ENI (private IP, SG port 22 only from Bastion SG)
```

### Why?
| Link | Function | Implementation |
|---|---|---|
| Public subnet + IGW route | Bastion reachable | Route 0.0.0.0/0 to IGW |
| Bastion EC2 | Controlled SSH jump host | Harden operating system, SSH keys, restricted administrator IP ranges |
| Bastion SG | Control inbound entry | TCP 22 only from authorized engineer IPs |
| Private EC2 SG | Restrict SSH to Bastion path | TCP 22 from Bastion SG |
| VPC-local routing + ENI | Deliver SSH to private IP | Local routes within VPC; no NAT required |

**Modern alternatives:** EC2 Instance Connect Endpoint offers private SSH access without public bastion; Systems Manager Session Manager uses IAM, SSM Agent and private endpoints or egress, usually without inbound SSH exposure. Bastion incurs operational patching and monitoring requirements; neither NAT Gateway nor ALB substitutes for SSH admin path.

## AWS Onion exam heuristic
1. **What must stay constant?** IPv4 identity → EIP.
2. **Who initiates communication?** Private EC2 to internet → public NAT Gateway and routes.
3. **What failure domain?** Multi-AZ egress → independent zonal NAT paths (or evaluate regional NAT).
4. **Who must enter the private network?** Remote operator → Bastion/EIC Endpoint/Session Manager.
5. **Which component performs which job?** IP identity (EIP), routing (Route Tables), translation (NAT), gateway (IGW), administration (Bastion/SSM), access control (SG/NACL).

**Critical mental rule:** HA of compute does not imply HA of every network dependency.

## Related
- [[EC2 - Networking and Connectivity]]
- [[NAT Gateway - Private to Public Functional Onion]]
- [[EC2 - Architecture Patterns and Auto Scaling]]
- [[AWS-Onion - Functional Onion]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[Systems Manager - Mental Model]]
