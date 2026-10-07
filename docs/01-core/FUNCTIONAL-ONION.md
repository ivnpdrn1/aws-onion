# AWS-Onion — Functional Onion Model

> **Functional Onion is the temporal axis: THROUGH time.**  
> Vertical Onion explains relationships BETWEEN layers; Horizontal Onion explains composition WITHIN a layer.

See [Vertical, Horizontal and Functional Axes](VERTICAL-HORIZONTAL-ONION.md).

## Purpose

AWS-Onion now uses two complementary views:

| View | Core question | What it explains |
|---|---|---|
| **Structural Onion** | **What layers/components exist and how are they related?** | Architecture, dependency, network path, authorization, placement |
| **Functional Onion** | **When an action occurs, what changes in sequence and what survives?** | Lifecycle, state transitions, persistence, side effects, recovery |

The Structural Onion is mainly **spatial/logical**.

The Functional Onion is mainly **temporal/causal**.

> **Structural Onion = WHAT is connected.**  
> **Functional Onion = WHAT HAPPENS NEXT.**

---

## Functional Onion — canonical sequence

Every Functional Onion should be read from the initiator through the target resource and then through the dependent/accessory blocks affected by the action.

~~~text
#1 INITIATOR
      ↓
#2 ACTION / CONTROL REQUEST
      ↓
#3 PRIMARY RESOURCE
      ↓
#4 RUNTIME / VOLATILE STATE
      ↓
#5 PERSISTENT STATE
      ↓
#6 NETWORK IDENTITY / CONNECTIVITY
      ↓
#7 UNDERLYING INFRASTRUCTURE
      ↑
#8 STATUS / OBSERVABILITY
      ↑
#9 NEXT DECISION
~~~

The numbering is conceptual. A block may be transversal rather than a literal packet hop.

---

## Primary block versus accessory blocks

A Functional Onion identifies one **primary resource** and the components whose behavior depends on the requested function.

For EC2 lifecycle:

| Block | Role |
|---|---|
| **EC2 instance** | Primary block whose lifecycle state is changing |
| **OS / RAM** | Runtime / volatile accessory |
| **EBS root + data volumes** | Persistent-storage accessory |
| **ENI / private IPv4 / IPv6 / public IPv4 / Elastic IP** | Network-identity accessories |
| **Physical host / platform** | Underlying infrastructure block |
| **Status checks / CloudWatch** | Evidence / observability block |

Mental rule:

> **An EC2 lifecycle action is not only an EC2 event. It can change the behavior of several attached or underlying blocks.**

---

## Functional Onion questions

For every AWS action, ask these questions in order:

| # | Question |
|---:|---|
| 1 | **Who or what initiates the function?** |
| 2 | **What action is requested?** |
| 3 | **What is the primary resource and target state?** |
| 4 | **What volatile/runtime state changes?** |
| 5 | **What persistent state remains, moves, or is deleted?** |
| 6 | **What happens to network identity/connectivity?** |
| 7 | **Does the underlying infrastructure remain relevant or change?** |
| 8 | **What status/evidence comes back?** |
| 9 | **What next transition is possible?** |

This produces a repeatable learning method:

> **TRIGGER → TARGET → CHANGE → PERSIST → RELEASE → EVIDENCE → NEXT STEP**

---

## Persistence vocabulary

Functional Onion diagrams should explicitly label each dependent block with one of these outcomes:

| Label | Meaning |
|---|---|
| **KEEP** | Resource/identity remains |
| **LOSE** | Volatile state disappears |
| **SAVE** | State is written into another persistent layer |
| **RESTORE** | Saved state is loaded back |
| **RELEASE** | AWS relinquishes the resource/assignment |
| **DELETE** | Resource lifecycle ends |
| **REATTACH** | Existing dependent resource is connected again |
| **MAY CHANGE** | Logical resource persists but underlying placement can differ |
| **NOT SPECIFIED** | The source lesson does not define the behavior |

This is especially important for certification study because it prevents assumptions where a lesson did not explicitly state a behavior.

---

# EC2 Lifecycle — first complete Functional Onion

## Phase 0 — Source image

~~~text
AMI
 ↓ Launch
EC2 PENDING
 ↓
EC2 RUNNING
~~~

**Primary block:** EC2  
**Function:** create/launch compute from an AMI.

---

## Phase 1 — Running baseline

~~~text
EC2 = RUNNING
OS / RAM = active
EBS = attached
Network identity = active
Physical host = hosting the logical instance
~~~

Running is the baseline from which Reboot, Stop, Hibernate and Terminate are compared.

---

## Phase 2 — Reboot Functional Onion

The source lesson defines Reboot as equivalent to an OS reboot, with DNS and IPv4/IPv6 retained and billing unaffected.

~~~text
ADMIN / CONTROL
      ↓ REBOOT
EC2 RUNNING
      ↓
OS RESTARTS
      │
      ├── DNS ............. KEEP
      ├── IPv4 ............ KEEP
      ├── IPv6 ............ KEEP
      └── BILLING ......... CONTINUES
      ↓
EC2 RUNNING
~~~

Core idea:

> **Reboot changes the runtime/OS while preserving the network identity described by the lesson.**

---

## Phase 3 — Stop Functional Onion

This is the clearest Functional Onion because several accessories behave differently.

~~~text
ADMIN / CONTROL
      ↓ STOP
EC2 RUNNING
      ↓
EC2 STOPPING
      ↓
EC2 STOPPED
      │
      ├── RAM ................. LOSE
      ├── EBS ................. KEEP + CHARGEABLE
      ├── Private IPv4 ........ KEEP
      ├── IPv6 ................ KEEP
      ├── Dynamic Public IPv4 . RELEASE
      ├── Elastic IP .......... KEEP
      └── Physical Host ....... MAY CHANGE on next Start
~~~

Then:

~~~text
EC2 STOPPED
      ↓ START
EC2 PENDING
      ↓
EC2 RUNNING
      │
      └── may execute on a different physical host
~~~

Core idea:

> **Stop preserves the logical machine while different accessory blocks follow different persistence rules.**

This directly demonstrates:

> **Compute lifecycle ≠ RAM lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.**

---

## Phase 4 — Hibernate Functional Onion

Hibernate is a special Stop path because the volatile RAM state is moved into persistent storage.

~~~text
ADMIN / CONTROL
      ↓ HIBERNATE
EC2 RUNNING
      ↓
RAM
      ↓ SAVE
EBS
      ↓
EC2 STOPPED
~~~

Resume:

~~~text
EC2 START
      ↓
EBS ROOT STATE ........ RESTORE
RAM CONTENTS .......... RESTORE
PROCESSES ............. RESUME
DATA VOLUMES .......... REATTACH
INSTANCE ID ........... KEEP
~~~

The lesson also states:

- applies to supported AMIs,
- hibernation must be enabled at launch,
- specific prerequisites apply.

Core idea:

> **Hibernate uses the Storage layer to preserve volatile Compute state.**

---

## Phase 5 — Terminate Functional Onion

~~~text
ADMIN / CONTROL
      ↓ TERMINATE
EC2 RUNNING
      ↓
SHUTTING-DOWN
      ↓
TERMINATED
      │
      ├── EC2 .............. DELETE
      ├── Root EBS ......... DELETE by default
      └── Return to Running  IMPOSSIBLE
~~~

Core idea:

> **Stop preserves the logical machine. Terminate ends the logical machine.**

---

## Phase 6 — Retire / Recover infrastructure branch

Retirement and recovery are different because the initiating condition can originate **below EC2**.

### Retire

~~~text
PHYSICAL HOST / HARDWARE
           X irreparable failure
           ↑
AWS detects condition
           ↑
scheduled retirement
           ↓
EC2 is STOPPED or TERMINATED by AWS
~~~

### Recover

~~~text
UNDERLYING HARDWARE / PLATFORM ISSUE
                ↑
        SYSTEM STATUS CHECK
                ↑
           CLOUDWATCH
                ↑
      ADMIN / AUTOMATION
                ↓ RECOVER
               EC2
~~~

The lesson states that the recovered instance is identical to the original instance.

Core idea:

> **A failure can originate downstream in infrastructure, travel upstream as evidence, and cause a new downstream corrective action.**

---

# Functional Onion — EC2 accessory survival matrix

| Function / phase | EC2 | RAM | EBS | Private IPv4 / IPv6 | Dynamic public IPv4 | Elastic IP | Physical host | Return path |
|---|---|---|---|---|---|---|---|---|
| **Running** | Active | Active | Attached | Active | Active if assigned | Active if associated | Current host | Already Running |
| **Reboot** | Returns to Running | OS/runtime restarts | **Not specified separately by lesson** | **KEEP** | **KEEP** as part of IPv4 retention | **Not specified separately** | **Not specified** | Running |
| **Stop** | Stopped | **LOSE** | **KEEP + chargeable** | **KEEP** | **RELEASE** | **KEEP** | **MAY CHANGE after Start** | Start → Pending → Running |
| **Hibernate** | Stopped/resumable | **SAVE → EBS** | **KEEP / restore** | **Not specified in lesson** | **Not specified** | **Not specified** | **Not specified** | Start → restore RAM/processes |
| **Terminate** | **DELETE** | Ends with instance | **Root EBS DELETE by default** | **Not specified in lesson** | **Not specified** | **Not specified** | Relationship ends | None |
| **Retire** | AWS stops or terminates | Depends on resulting action | Depends on resulting action | Depends on resulting action | Depends | Depends | **Underlying host failure** | Depends |
| **Recover** | Recovered | **Not detailed in lesson** | **Not detailed in lesson** | **Not detailed in lesson** | **Not detailed** | **Not detailed** | Hardware/platform impairment addressed | Recovered instance |

---

# Functional Onion — one-line sequence

~~~text
LAUNCH
AMI → Pending → Running

REBOOT
Running → OS reboot → Running
         KEEP DNS / IPv4 / IPv6

STOP
Running → Stopping → Stopped
          LOSE RAM
          KEEP EBS
          KEEP private IPv4 / IPv6
          RELEASE dynamic public IPv4
          KEEP Elastic IP
          MAY CHANGE physical host
             ↓
           Start
             ↓
          Pending → Running

HIBERNATE
Running → RAM SAVE to EBS → Stopped
                           ↓ Start
                   RESTORE RAM + processes
                           ↓
                        Running

TERMINATE
Running → Shutting-down → Terminated
                          DELETE EC2
                          DELETE root EBS by default
                          NO RETURN

RETIRE / RECOVER
Physical/platform issue
          ↑ evidence
CloudWatch / status
          ↓ corrective action
EC2 stop / terminate / recover
~~~

---

## Functional Onion master rule

> **Do not memorize only the state name. Follow the function through every dependent block.**

For lifecycle questions, reconstruct the answer as:

~~~text
ACTION
  ↓
EC2 STATE
  ↓
RAM
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

This is the **Functional Onion**: a time-based view of AWS interdependence.

Related:
- [Core AWS-Onion Mental Model](aws-onion-core.md)
- [AWS-Onion Downstream Model](DOWNSTREAM-MODEL.md)
- [EC2 Instance Lifecycle through AWS-Onion](../02-compute-networking/ec2-instance-lifecycle.md)
