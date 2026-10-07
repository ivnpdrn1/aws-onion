---
type: concept
domain: compute
tags: [aws, ec2, lifecycle, ebs, networking, cloudwatch, aws-onion, tables]
---

# EC2 — Instance Lifecycle

> [!abstract] Study method
> Review in sequence: **State → Transition → Persistence → Cost → Infrastructure → AWS-Onion layer**.

## 1 — State / action / condition

| Concept | Type | Essential meaning |
|---|---|---|
| Pending | State | Preparing instance |
| Running | State | Instance active |
| Rebooting | Transition | OS reboot in progress |
| Stopping | State | Transition toward Stopped |
| Stopped | State | Compute stopped; can Start again |
| Shutting-down | State | Termination in progress |
| Terminated | State | Deleted; cannot recover |
| Stop | Action | Stop EBS-backed compute temporarily |
| Hibernate | Action | Stop while preserving RAM in EBS |
| Reboot | Action | Restart guest OS |
| Terminate | Action | Delete instance |
| Retire | Infrastructure condition | AWS hardware failure leads to scheduled stop/terminate |
| Recover | Corrective action | Recover from underlying hardware/platform impairment |

## 2 — Transition table

| From | Action | Through | To |
|---|---|---|---|
| AMI | Launch | Pending | Running |
| Running | Reboot | Rebooting | Running |
| Running | Stop | Stopping | Stopped |
| Stopped | Start | Pending | Running |
| Running | Hibernate | Stopping | Stopped / resumable |
| Running | Terminate | Shutting-down | Terminated |

## 3 — Resource persistence

| Operation | RAM | EBS | Private IPv4 / IPv6 | Public IPv4 | Elastic IP |
|---|---|---|---|---|---|
| Reboot | OS reboot | No deletion described | Retained | Retained | Not separately discussed |
| Stop / Start | **Lost** | **Retained + chargeable** | **Retained** | **Released** | **Retained** |
| Hibernate | **Saved to EBS / restored** | Root restored; data volumes reattached | Not specified in lesson | Not specified | Not specified |
| Terminate | Instance deleted | **Root deleted by default** | Not specified | Not specified | Not specified |

## 4 — Reboot vs Stop vs Hibernate vs Terminate

| Question | Reboot | Stop | Hibernate | Terminate |
|---|---|---|---|---|
| Purpose | Restart OS | Pause compute | Preserve RAM and pause | Delete instance |
| Return? | Yes | Yes | Yes | **No** |
| RAM | OS reboot | **Lost** | **Saved/restored** | Ends with instance |
| EBS | Remains | **Remains** | **Remains** | **Root deleted by default** |
| Billing | **Unaffected** | No EC2 charge while stopped | Compute stopped | Ends |
| Key phrase | Restart | Pause | Preserve memory | Delete |

## 5 — STOP deep comparison

| Layer | Behavior |
|---|---|
| Compute | Stops; can Start again |
| RAM | **Lost** |
| EBS | **Retained; chargeable** |
| Private IPv4 | **Retained** |
| IPv6 | **Retained** |
| Dynamic public IPv4 | **Released** |
| Elastic IP | **Retained** |
| Physical host | **Can change after Start** |

> [!tip]
> **Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.**

## 6 — HIBERNATE deep comparison

| Item | Behavior |
|---|---|
| Supported AMI | Required |
| Enabled at launch | Required |
| RAM | Saved to EBS |
| Root EBS | Restored |
| RAM after Start | Reloaded |
| Processes | Resume |
| Data volumes | Reattached |
| Instance ID | Retained |

> **Hibernate = RAM → EBS → RAM → processes resume.**

## 7 — RETIRE vs RECOVER

| | Retire | Recover |
|---|---|---|
| Cause | Irreparable underlying hardware failure | Underlying hardware/platform impairment |
| Layer | Physical/platform | Physical/platform |
| AWS behavior | Scheduled Stop or Terminate | Recovery |
| CloudWatch/system status | Not the main retirement point | **Can be monitored** |
| Result | Depends on stop/terminate | Instance described as identical to original |

## 8 — Exam matrix

| Concept | One-line memory |
|---|---|
| Reboot | **OS reboot; addresses retained; billing continues** |
| Stop | **RAM lost; EBS stays; private IPs stay; public IPv4 goes** |
| Hibernate | **RAM saved to EBS; processes resume** |
| Retire | **Underlying host hardware failure** |
| Terminate | **Delete; cannot recover; root EBS deleted by default** |
| Recover | **Hardware/platform issue + system status + CloudWatch** |

## 9 — AWS-Onion layer map

| # | Layer | Lifecycle question |
|---:|---|---|
| 1 | User / Admin / Automation | Who initiated the action? |
| 2 | AWS Control / Management | Which EC2 action was requested? |
| 3 | Compute | What EC2 state changed? |
| 4 | OS / RAM | What happened to runtime state? |
| 5 | EBS | What persisted in storage? |
| 6 | Network identity | Which addresses persisted or changed? |
| 7 | Physical / platform | Did the underlying host matter? |
| 8 | Status / CloudWatch | What evidence came back? |

> **↓ Actions descend. ↑ Evidence returns.**

> **STOP preserves the logical machine. TERMINATE ends it.**

## Related
- [[AWS-Onion - Functional Onion]]
- [[AWS-Onion - Mental Model]]
- [[EC2 - Networking and Connectivity]]
- [[CloudWatch - Mental Model]]
