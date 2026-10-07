# EC2 Instance Lifecycle through AWS-Onion

## Purpose

This section organizes the EC2 lifecycle as a **sequence of comparison tables** so the concepts can be reviewed from several angles without losing the relationships among layers.

## Functional Onion interpretation

EC2 Lifecycle is the first complete example of the AWS-Onion **Functional Onion**: instead of looking only at which layers exist, we follow one lifecycle action through the EC2 and its dependent/accessory blocks.

~~~text
ACTION
  ↓
EC2 STATE
  ↓
OS / RAM
  ↓
EBS
  ↓
NETWORK IDENTITY
  ↓
PHYSICAL HOST
  ↑
STATUS / OBSERVABILITY
  ↑
NEXT ACTION
~~~

For every phase, the study question becomes:

> **What happens to EC2 itself, and which accessories are KEPT, LOST, SAVED, RESTORED, RELEASED, DELETED, REATTACHED, or MAY CHANGE?**

See [AWS-Onion — Functional Onion Model](../01-core/FUNCTIONAL-ONION.md).

> **Study order:** State → Transition → Persistence → Cost/Recovery → Infrastructure condition → AWS-Onion meaning.

---

## Table 1 — First distinction: state, action, or infrastructure condition?

This is the most important organizational distinction.

| Concept | Type | What it represents | Key point |
|---|---|---|---|
| **Pending** | EC2 state | Instance is being prepared after launch/start | Transitional state before Running |
| **Running** | EC2 state | Instance is active | Normal operating state |
| **Rebooting** | Transitional lifecycle condition | OS-level reboot in progress | Returns to Running |
| **Stopping** | EC2 state | Instance is transitioning toward Stopped | Used by Stop and Hibernate flow |
| **Stopped** | EC2 state | Compute is stopped but instance can be started again | Persistent resources can remain |
| **Shutting-down** | EC2 state | Termination is in progress | Leads to Terminated |
| **Terminated** | EC2 state | Instance has been deleted | Cannot be recovered |
| **Stop** | Action | Requests a running EBS-backed instance to stop | RAM is lost; EBS remains |
| **Hibernate** | Action | Stops while saving RAM contents to EBS | Requires supported AMI and launch-time enablement |
| **Reboot** | Action | Reboots the guest operating system | Addresses are retained; billing unaffected |
| **Terminate** | Action | Deletes the EC2 instance | Root EBS deleted by default |
| **Retire** | AWS infrastructure event | AWS schedules stop/termination after irreparable host hardware failure | Originates below the guest instance |
| **Recover** | Corrective action | Recovery for impairment caused by underlying hardware/platform | CloudWatch can monitor system status checks |

> **Mental rule:** A lifecycle **state** is where the instance is. An **action** moves it. A retirement/recovery condition can originate from the AWS infrastructure below it.

---

## Table 2 — Core state-transition sequence

| # | Starting point | Action / cause | Transitional state | Result | Can continue? |
|---:|---|---|---|---|---|
| 1 | **AMI** | Launch | **Pending** | **Running** | Yes |
| 2 | **Running** | Reboot | **Rebooting** | **Running** | Yes |
| 3 | **Running** | Stop | **Stopping** | **Stopped** | Yes → Start |
| 4 | **Stopped** | Start | **Pending** | **Running** | Yes |
| 5 | **Running** | Hibernate | **Stopping** | **Stopped** | Yes → Start/resume |
| 6 | **Running** | Terminate | **Shutting-down** | **Terminated** | **No** |

~~~text
AMI
 ↓ Launch
PENDING
 ↓
RUNNING
 ├─ Reboot ─────→ REBOOTING ─────→ RUNNING
 ├─ Stop ───────→ STOPPING ───────→ STOPPED ─→ Start → PENDING → RUNNING
 ├─ Hibernate ──→ STOPPING ───────→ STOPPED ─→ Start / resume
 └─ Terminate ──→ SHUTTING-DOWN ──→ TERMINATED
~~~

> **Exam anchor:** **Terminated is final.**

---

## Table 3 — What changes and what persists?

This table compares only details explicitly presented in the lesson. Where the lesson does not state a behavior, it is marked accordingly.

| Operation | RAM | EBS | Private IPv4 / IPv6 | Public IPv4 | Elastic IP | Instance ID / processes |
|---|---|---|---|---|---|---|
| **Reboot** | Lesson frames this as an OS reboot | No deletion described | **Retained** | **Retained** as part of IPv4 retention | Not separately discussed | OS restarts; returns to Running |
| **Stop / Start** | **Lost** | **Remains attached; chargeable** | **Retained** | **Released** | **Retained** | Instance can be started again |
| **Hibernate / Resume** | **Saved to EBS, then reloaded** | Root volume restored to previous state; attached data volumes reattached | Not specified in this lesson | Not specified in this lesson | Not specified in this lesson | **Processes resume; instance ID retained** |
| **Terminate** | Not specifically discussed; instance is deleted | **Root EBS deleted by default** | Not specified in this lesson | Not specified in this lesson | Not specified in this lesson | **Instance cannot be recovered** |
| **Recover** | Not detailed in this lesson | Not detailed in this lesson | Not detailed in this lesson | Not detailed in this lesson | Not detailed in this lesson | **Recovered instance is described as identical to original** |

---

## Table 4 — Compute charge, storage charge, and return path

| Operation / state | EC2 compute charge | EBS charge | Can return to Running? | Essential lesson |
|---|---|---|---|---|
| **Running** | Yes | Yes if EBS is used | Already Running | Active compute |
| **Rebooting** | **Billing unaffected** | Continues | **Yes** | Reboot is not Stop |
| **Stopped** | **No EC2 instance charge** | **Yes — EBS remains chargeable** | **Yes** | Compute cost and storage cost are separate |
| **Hibernate / stopped** | Compute is stopped | EBS is still required to preserve state | **Yes** | RAM persistence is moved into EBS |
| **Terminated** | No | Root EBS deleted by default | **No** | Lifecycle ends |

> **AWS-Onion rule:** **Compute billing ≠ Storage billing.**

---

## Table 5 — Reboot vs Stop vs Hibernate vs Terminate

This is the fastest table for exam review.

| Question | **Reboot** | **Stop** | **Hibernate** | **Terminate** |
|---|---|---|---|---|
| Primary purpose | Restart OS | Stop compute temporarily | Stop while preserving RAM state | Delete instance |
| Returns to same logical workload? | Yes | Yes after Start | Yes after resume | **No** |
| RAM preserved? | Lesson does not frame it as persistence | **No** | **Yes — saved to EBS** | Not relevant after deletion |
| EBS remains? | Yes / no deletion described | **Yes** | **Yes** | **Root EBS deleted by default** |
| Private IPv4 / IPv6 | **Retained** | **Retained** | Not specified here | Not specified here |
| Public IPv4 | **Retained** | **Released** | Not specified here | Not specified here |
| Elastic IP | Not separately discussed | **Retained** | Not specified here | Not specified here |
| Billing | **Unaffected** | No EC2 compute charge while stopped | Compute stopped; EBS required | Instance deleted |
| Different physical host after return? | Not stated | **Yes — can be migrated to a different host** | Not stated | Not applicable |
| Key phrase | **Restart** | **Pause compute** | **Preserve memory** | **Delete** |

> **Memory phrase:**  
> **Reboot = restart. Stop = pause and return. Hibernate = preserve memory and return. Terminate = delete and never return.**

---

## Table 6 — STOP in detail

| Layer / resource | What happens on Stop / Start? | Why it matters in AWS-Onion |
|---|---|---|
| **Compute** | Instance stops; later Start returns through Pending → Running | Compute has its own lifecycle |
| **RAM** | **Lost** | Volatile state belongs to the runtime layer |
| **EBS** | **Remains attached and chargeable** | Storage persists independently of compute |
| **Private IPv4** | **Retained** | Part of persistent network identity |
| **IPv6** | **Retained** | Network identity can outlive the running compute state |
| **Dynamic public IPv4** | **Released** | Public addressing can have a different lifecycle |
| **Elastic IP** | **Retained** | Static public addressing behaves differently from dynamic public IPv4 |
| **Physical host** | Instance can start on a **different host** | Logical EC2 identity is abstracted from physical hardware |

> **Core AWS-Onion lesson:**  
> **Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.**

---

## Table 7 — HIBERNATE in detail

| Requirement / result | Lesson detail | AWS-Onion interpretation |
|---|---|---|
| Supported AMI | Required | Capability depends on the image/platform combination |
| Enabled at launch | **Must be enabled when launched** | Lifecycle capabilities can be decided at creation time |
| RAM | **Saved to EBS** | Volatile Compute state crosses into persistent Storage |
| Root EBS | Restored to previous state | Storage becomes the source for runtime reconstruction |
| RAM on Start | **Reloaded** | Runtime state is reconstructed |
| Processes | **Previously running processes resume** | Higher-level execution state returns |
| Data volumes | **Previously attached volumes reattached** | Storage relationships are restored |
| Instance ID | **Retained** | Logical instance identity persists |

~~~text
COMPUTE / RAM
      ↓ save
EBS STORAGE
      ↓ restore
COMPUTE / RAM
      ↓
PROCESSES RESUME
~~~

> **Hibernate = Storage preserves volatile Compute state.**

---

## Table 8 — REBOOT in detail

| Aspect | Lesson detail | Meaning |
|---|---|---|
| Type of operation | Equivalent to **OS reboot** | Primarily a guest/runtime operation |
| DNS name | **Retained** | Instance identity is not recreated |
| IPv4 | **Retained** | Network identity remains |
| IPv6 | **Retained** | Network identity remains |
| Billing | **Unaffected** | Reboot is not a stopped state |
| Final state | Returns to **Running** | Temporary operational restart |

> **AWS-Onion reading:** Reboot is comparatively shallow: the OS/runtime changes while the surrounding EC2 architecture remains intact.

---

## Table 9 — RETIRE vs RECOVER

These concepts are best studied together because both involve the infrastructure **underneath** the guest instance.

| Question | **Retire** | **Recover** |
|---|---|---|
| Trigger | AWS detects **irreparable failure of underlying hardware** | Instance is impaired because of **underlying hardware/platform issues** |
| Origin layer | Physical / platform infrastructure | Physical / platform infrastructure |
| AWS action | Schedules instance retirement | Recovery mechanism can restore the instance |
| At scheduled time | Instance is **stopped or terminated by AWS** | Recovered instance is described as **identical to original** |
| Monitoring | Not the emphasis of retirement slide | **CloudWatch can monitor system status checks** |
| AWS-Onion direction | Failure originates deep in the Onion and propagates upward as an operational condition | Observation can trigger a corrective action back downstream |

~~~text
PHYSICAL / PLATFORM PROBLEM
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

> **Key distinction:** The EC2 guest may be healthy while the host underneath it has the problem.

---

## Table 10 — TERMINATE in detail

| Aspect | What the lesson states | Consequence |
|---|---|---|
| Meaning | **Delete the EC2 instance** | Logical machine ends |
| Recovery | **Cannot recover a terminated instance** | Final state |
| Root EBS | **Deleted by default** | Root storage normally ends with the instance |
| Return to Running | **No** | Must create/launch another instance instead |

> **STOP preserves the logical machine. TERMINATE ends its lifecycle.**

---

## Table 11 — AWS-Onion: which layer is emphasized by each operation?

| Operation / condition | Primary layer emphasized | Secondary layer(s) revealed | AWS-Onion lesson |
|---|---|---|---|
| **Launch / Pending** | Compute creation | AMI / control plane | A logical machine is constructed from multiple AWS dependencies |
| **Reboot** | OS / runtime | Network identity, billing | A shallow operation can leave deeper layers unchanged |
| **Stop** | Compute | RAM, EBS, network, host | Different layers have different persistence rules |
| **Hibernate** | Compute ↔ Storage | Processes, instance identity | Storage can preserve volatile runtime state |
| **Retire** | Physical infrastructure | Compute lifecycle | Logical EC2 is separate from underlying hardware |
| **Terminate** | Compute lifecycle | Root storage | Deletion can propagate into dependent resources |
| **Recover** | Observability + infrastructure | Compute | Evidence moves upward; corrective action moves downward |

---

## Table 12 — Final exam-review matrix

| Concept | Remember this | Do not confuse with |
|---|---|---|
| **Pending** | Preparing instance | Running |
| **Running** | Active instance | Pending/Stopped |
| **Reboot** | OS reboot; addresses retained; billing unaffected | Stop |
| **Stop** | RAM lost; EBS remains; private IPs retained; public IPv4 released | Hibernate |
| **Hibernate** | RAM saved to EBS and restored | Ordinary Stop |
| **Stopped** | No EC2 compute charge; EBS still chargeable | Terminated |
| **Retire** | AWS underlying hardware failure | User-requested Terminate |
| **Terminate** | Delete instance; cannot recover; root EBS deleted by default | Stop |
| **Recover** | Hardware/platform impairment + system status monitoring | Reboot |

---

## AWS-Onion master model

~~~text
#1 USER / ADMINISTRATOR / AUTOMATION
        ↓
#2 AWS CONTROL / MANAGEMENT
        ↓
#3 COMPUTE STATE
        ↓
#4 OS / RAM
        ↓
#5 EBS STORAGE
        ↓
#6 NETWORK IDENTITY
        ↓
#7 AWS PHYSICAL / PLATFORM INFRASTRUCTURE
        ↑
#8 STATUS / CLOUDWATCH / EVIDENCE
        ↑
#1 NEXT ADMINISTRATOR / AUTOMATION DECISION
~~~

> **↓ Actions descend. ↑ Evidence returns.**

> **AWS-Onion asks three questions for every lifecycle operation:**  
> **What changed? What persisted? Which layer owns that behavior?**

Related:
- [AWS-Onion Downstream Model](../01-core/DOWNSTREAM-MODEL.md)
- [EC2 Networking & Connectivity](ec2-networking.md)
- [CloudWatch](../04-operations/cloudwatch.md)
