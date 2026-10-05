---
type: concept
domain: compute
tags: [aws, ec2, lifecycle, ebs, networking, cloudwatch, aws-onion]
---

# EC2 — Instance Lifecycle

## 1. State map

~~~text
AMI
 ↓
PENDING
 ↓
RUNNING
 ├─ Reboot ─────────→ REBOOTING ─→ RUNNING
 ├─ Stop ───────────→ STOPPING ───→ STOPPED ─→ Start → PENDING → RUNNING
 ├─ Hibernate ──────→ STOPPING ───→ STOPPED ─→ Resume
 └─ Terminate ──────→ SHUTTING-DOWN → TERMINATED
~~~

> **Terminated = cannot be recovered.**

## 2. STOP

- EBS-backed instances only.
- EC2 compute charge stops.
- EBS remains attached and chargeable.
- RAM is lost.
- Private IPv4 + IPv6 retained.
- Dynamic public IPv4 released.
- Elastic IP retained.
- Start can place the instance on a different host.

> **STOP exposes abstraction: logical EC2 ≠ permanent physical host.**

## 3. HIBERNATE

- Supported AMIs only.
- Must be enabled at launch.
- RAM is saved to EBS.
- On resume:
  - root EBS state restored,
  - RAM reloaded,
  - processes resume,
  - data volumes reattached,
  - instance ID retained.

~~~text
RAM → EBS → RAM
          ↓
     processes resume
~~~

> **Hibernate = Storage preserves volatile Compute state.**

## 4. REBOOT

- Equivalent to OS reboot.
- DNS retained.
- IPv4 and IPv6 retained.
- Billing unaffected.

> **Reboot mainly affects OS/runtime, not the surrounding EC2 identity.**

## 5. RETIRE

- AWS detects irreparable underlying hardware failure.
- At scheduled retirement, AWS stops or terminates the instance.

> **Retirement = Physical layer problem, not necessarily an EC2/OS problem.**

## 6. TERMINATE

- Deletes the EC2 instance.
- Cannot recover a terminated instance.
- Root EBS deleted by default.

~~~text
STOP      → can return
TERMINATE → cannot return
~~~

## 7. RECOVER

- CloudWatch can monitor system status.
- Applies to underlying hardware/platform impairment.
- Recovered instance is intended to be identical to the original.

~~~text
HARDWARE / PLATFORM
       ↑
SYSTEM STATUS
       ↑
CLOUDWATCH
       ↑
ADMIN / AUTOMATION
       ↓
RECOVERY
       ↓
EC2
~~~

## 8. One-table memory view

| Operation | RAM | EBS | Network | Host | Return? |
|---|---|---|---|---|---|
| Reboot | Restarted | Retained | Addresses retained | Same running environment | Yes |
| Stop / Start | Lost | Retained | Private IPv4/IPv6 retained; public IPv4 released; EIP retained | Can change | Yes |
| Hibernate | Saved/restored | Retained | Identity retained | Runtime reconstructed | Yes |
| Retire | Depends on stop/terminate | Depends | Depends | Underlying hardware failed | Depends |
| Terminate | Lost | Root deleted by default | Released with instance | Ends | No |
| Recover | Recovered by EC2 mechanism | Preserved | Identity preserved | Hardware/platform issue bypassed | Yes |

## 9. AWS-Onion mental model

~~~text
#1 ADMIN / AUTOMATION
        ↓
#2 EC2 CONTROL
        ↓
#3 COMPUTE STATE
        ↓
#4 OS / RAM
        ↓
#5 EBS
        ↓
#6 ENI / IP
        ↓
#7 PHYSICAL HOST
        ↑
#8 STATUS / CLOUDWATCH
        ↑
   ADMIN / AUTOMATION
~~~

## Memory anchors

> **↓ Actions descend. ↑ Evidence returns.**

> **Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.**

> **STOP preserves the logical machine. TERMINATE ends it.**

## Related
- [[AWS-Onion - Mental Model]]
- [[EC2 - Networking and Connectivity]]
- [[CloudWatch - Mental Model]]
