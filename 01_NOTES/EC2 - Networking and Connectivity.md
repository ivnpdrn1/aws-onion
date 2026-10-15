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

## Architecture patterns: Spread vs ENA vs Nitro (2026-10-15)

| Need | AWS choice | Why it fits | Why not the other choices |
|---|---|---|---|
| Minimize correlated physical host failure for a small group of critical EC2 | **Spread Placement Group** | Places separate instances on distinct underlying hardware | Cluster optimizes proximity; Partition isolates sets of instances, not each individual instance |
| Enhanced conventional TCP/IP networking | **ENA**, with supported EC2 type and driver | Efficient, high-throughput IP networking | ENI provides network identity but not necessarily enhanced networking; EFA is specialized for supported HPC communication |
| Near bare-metal virtual-machine efficiency | **EC2 built on AWS Nitro System** | Lightweight hypervisor plus specialized offload for I/O and security | Dedicated Host concerns tenancy and hardware control, not the general virtualization architecture |

**Additional HPC requirement:** A Nitro-based instance does not automatically support EFA. If EFA is required, explicitly choose an EFA-compatible instance type and supporting AMI/software; often pair it with Cluster Placement Group for tightly coupled HPC.

### Vertical / Horizontal / Functional Onion

- **Physical placement:** Spread separates *individual* instances to reduce (not eliminate) correlated host failures; use multi-AZ architecture for AZ failure resilience.
- **Network technology:** ENI is the network interface identity. ENA accelerates ordinary IP networking. EFA provides specialized OS-bypass communication for compatible HPC applications; these concepts are not always mutually exclusive.
- **Virtualization substrate:** Nitro is the system enabling efficient EC2 virtualization and hardware offload. It is not equivalent to EFA or a Dedicated Host.

```text
Requirement → decide fault domain → Spread / Partition / Cluster
Requirement → decide networking mode → ENA (IP) / EFA (HPC)
Requirement → decide compute substrate → Nitro-compatible EC2 family
Then verify instance-type compatibility, AZ capacity, AMI driver and SG.
```

**Exam recall:** Spread = separate machines; ENA = accelerated IP; EFA = HPC OS-bypass; Nitro = EC2 underlying virtualization and hardware offload.

Related: [[EC2 - Nitro System and Nitro Enclaves]], [[EC2 - Architecture Patterns and Auto Scaling]], [[AWS-Onion - Functional Onion]].

## Placement Groups

- **Cluster** → together → performance.
- **Spread** → separate individual instances → resilience.
- **Partition** → separate groups → distributed systems.

## SSH Key Pair vs IAM Access Key — authentication boundaries (2026-10-15)

| Question | EC2 SSH Key Pair | AWS Access Key ID + Secret Access Key |
|---|---|---|
| Who authenticates you? | SSH server inside EC2 guest OS | AWS API authentication (Signature Version 4) |
| What is the secret? | SSH private key (often .pem) | Secret Access Key (often with session token for temporary creds) |
| What is public / identifier? | SSH public key authorized on EC2 | Access Key ID identifies AWS credential |
| What does it authorize? | Linux login as configured user, subject to OS permissions | AWS API requests subject to IAM policy, permissions boundaries, SCPs, etc. |
| Used for? | `ssh -i key.pem ec2-user@IP` | `aws ec2 describe-instances`, SDKs and AWS API calls |

**Functional Onion — SSH:** user SSH client/private key → network/IGW/routing/SG/NACL → EC2 ENI → sshd in guest → SSH public-key authentication → OS authorization.

**Functional Onion — API:** CLI/SDK → IAM credentials sign request → AWS API authenticates caller and checks permissions → service action.

**Do not conflate:** Access keys are **not** SSH key pairs. A private SSH key never needs to be sent to EC2. An EC2 instance does not require an SSH key pair if managed using SSM Session Manager or other authorized methods. For code running on EC2, **prefer an IAM role (instance profile) with temporary credentials**, not hard-coded IAM access keys.

**Exam:** Amazon Linux 2 traditional SSH authentication → **key pair**; AWS API with CLI → AWS credentials / IAM role, not SSH key pair.

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
