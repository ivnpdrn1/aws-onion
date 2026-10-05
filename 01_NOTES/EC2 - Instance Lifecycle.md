---
type: concept
domain: compute
tags: [aws, ec2, lifecycle, ebs, networking, cloudwatch, aws-onion]
---

# EC2 — Instance Lifecycle

## AWS-Onion idea

> **An EC2 lifecycle action descends through the architecture; its effects and health signals can return upward as operational evidence.**

~~~text
#1 Administrator / Automation
        ↓
#2 EC2 Control / Management
        ↓
#3 EC2 Compute State
        ↓
#4 OS / RAM
        ↓
#5 EBS Storage
        ↓
#6 ENI / IP addressing
        ↓
#7 Underlying AWS host/platform
        ↑
#8 Status / CloudWatch
        ↑
   Administrator
~~~

## Lifecycle

~~~text
AMI
 ↓
PENDING
 ↓
RUNNING
 ├─ Reboot ─────────────→ RUNNING
 ├─ Stop → STOPPING ────→ STOPPED → Start → PENDING → RUNNING
 ├─ Hibernate → STOPPING → STOPPED → Resume
 └─ Terminate → SHUTTING-DOWN → TERMINATED
~~~

## Memory anchors

> **STOP preserves the logical machine. TERMINATE ends it.**

> **Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.**

> **↓ Downstream = intent / actions / configuration.**  
> **↑ Upstream = status / metrics / errors / evidence.**

## Layer persistence

| Operation | RAM | EBS | Private IPv4 / IPv6 | Ordinary Public IPv4 | Elastic IP | Physical host |
|---|---|---|---|---|---|---|
| Reboot | rebooted | retained | retained | retained | retained | running environment remains |
| Stop / Start | lost | retained | retained | released | retained | may change |
| Hibernate / Resume | saved to EBS, then restored | retained | retained | lifecycle rules apply | retained | execution resumes from saved state |
| Terminate | lost | root EBS deleted by default | released with instance | released | no longer attached to terminated instance | relationship ends |

## Why Stop/Start matters

~~~text
Logical EC2
   ↓
Host A

STOP / START

Logical EC2
   ↓
Host B
~~~

The instance can remain logically “the same” while the physical host changes.

That is AWS abstraction in practice.

## Related
- [[AWS-Onion - Mental Model]]
- [[EC2 - Networking and Connectivity]]
- [[CloudWatch - Mental Model]]
