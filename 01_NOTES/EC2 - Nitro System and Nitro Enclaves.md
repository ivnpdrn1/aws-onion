---
type: exam-note
domain: compute
service: EC2
tags: [aws, ec2, nitro, nitro-enclaves, certification]
---

# EC2 — Nitro System and Nitro Enclaves

> [!summary]
> **Nitro = performance through specialization.**  
> **Enclave = security through isolation.**

## Vertical Onion

```text
APPLICATION / WORKLOAD
        ↓
EC2 INSTANCE
        ↓
NITRO SYSTEM
        ↓
PHYSICAL AWS INFRASTRUCTURE
```

## Horizontal Onion — Nitro

```text
NITRO SYSTEM
   ├── Nitro Hypervisor
   ├── Nitro Card for VPC
   ├── Nitro Card for EBS
   ├── Nitro Card for instance storage
   ├── Nitro Controller
   ├── Nitro Security Chip
   └── Nitro Enclaves
```

## Nitro Enclave memory

```text
HIGHLY SENSITIVE DATA
        ↓
ISOLATED COMPUTE
        ↓
NO persistent storage
NO interactive access
NO external networking
        ↓
CRYPTOGRAPHIC ATTESTATION
        ↓
KMS
```

## Exam triggers

| Trigger | Answer |
|---|---|
| underlying EC2 platform | Nitro System |
| near bare-metal performance | Nitro |
| specialized hardware/modules | Nitro |
| direct physical server access | Bare Metal |
| isolated sensitive compute | Nitro Enclaves |
| cryptographic attestation | Nitro Enclaves |
| no external networking | Nitro Enclaves |
| KMS + isolated processing | Nitro Enclaves |

> [!warning]
> Near bare-metal performance does **not** mean the instance is bare metal.

## Related
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[EC2 - Networking and Connectivity]]
- [[EC2 - Instance Lifecycle]]
