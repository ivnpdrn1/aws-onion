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

### Choosing ENI, ENA or EFA for HPC

| Component | Role | When |
|---|---|---|
| ENI | Virtual network interface with IP addresses and security groups | EC2 connectivity to a VPC |
| ENA | Enhanced Networking for high-performance ordinary IP traffic | Web services, databases and high-throughput TCP/IP |
| EFA | Specialized high-performance interface with OS-bypass for supported HPC software | Tightly coupled HPC and distributed training requiring low-latency inter-node communication |
| Cluster Placement Group | Places EC2 instances near each other in one Availability Zone | Complements EFA by optimizing placement |

These are not three mutually exclusive alternatives. EFA interfaces may provide ENA functionality for ordinary IP networking; EFA-only interfaces support specialized traffic without conventional IP networking.

**Two functional paths:**
```text
Conventional: Application -> OS TCP/IP stack -> ENA -> AWS network
HPC:          HPC application -> MPI/libfabric -> EFA OS-bypass -> AWS network
```

**Exam cue:** tightly coupled HPC + low latency between nodes = Cluster Placement Group + EFA (when supported). Cluster optimizes placement; EFA optimizes communication. ENI provides connectivity, not the specialized HPC transport.

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
