---
type: concept
domain: compute-networking
tags: [aws, ec2, eni, ssh, networking]
---

# EC2 — Networking and Connectivity

## User Data vs Metadata

**User Data** tells EC2 what to do at launch.  
**Instance Metadata** lets EC2 discover information about itself.

IMDS uses the link-local address `169.254.169.254`; IMDSv2 uses a token.

## EC2 status checks

- **System Status Check** → AWS underlying infrastructure — “building okay?”
- **Instance Status Check** → guest OS/instance — “apartment okay?”

`2/2 checks passed` means both are healthy.

## ENI / ENA / EFA

> **ENI connects. ENA accelerates. EFA coordinates clusters.**

The private IP belongs to the ENI. The ENI is the virtual NIC connecting the instance to the VPC.

## Placement Groups

- **Cluster** → together → performance.
- **Spread** → separate individual instances → resilience.
- **Partition** → separate groups → distributed systems.

## Administration paths

```text
SSH
Admin → network → SG:22 → ENI → EC2 → sshd

EC2 Instance Connect Endpoint
Admin → EIC → Endpoint → private VPC path → ENI → EC2 → SSH

Session Manager
Admin → Systems Manager → SSM control path → Agent → EC2
```

> [!tip]
> EIC Endpoint = “I want SSH, but the EC2 is private.”  
> Session Manager = “I want administration without opening inbound SSH.”

## Linux view

`ifconfig`, `ip addr`, `ip link` and `ip route` show the guest OS network view. They do **not** show AWS Route Tables, NACLs or Security Groups.

## Related
- [[AWS-Onion - Mental Model]]
- [[Systems Manager - Mental Model]]
- [[CloudWatch - Mental Model]]
