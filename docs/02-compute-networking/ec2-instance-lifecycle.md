# EC2 Instance Lifecycle through AWS-Onion

## 1. The big picture — EC2 is a state machine

The lesson is easier to understand if we first see the instance as moving through **states**.

~~~text
AMI
 │
 │ Launch
 ▼
PENDING
 │
 ▼
RUNNING
 │
 ├──────── Reboot ────────→ REBOOTING ────────→ RUNNING
 │
 ├──────── Stop ──────────→ STOPPING ──────────→ STOPPED
 │                                             │
 │                                             └─ Start → PENDING → RUNNING
 │
 ├──────── Hibernate ─────→ STOPPING ──────────→ STOPPED
 │                                             │
 │                                             └─ Start → resume
 │
 └──────── Terminate ─────→ SHUTTING-DOWN ─────→ TERMINATED
~~~

A terminated instance cannot be recovered.

### AWS-Onion reading

> **The EC2 state is only the Compute layer. A lifecycle action can also affect RAM, EBS, network addressing and the physical host underneath.**

---

## 2. STOPPING an EC2 instance

### What the lesson says

- Applies to **EBS-backed instances**.
- A stopped instance does not incur the EC2 compute charge.
- The attached **EBS volumes remain attached and chargeable**.
- Data in **RAM is lost**.
- After Stop/Start, the instance can run on a different underlying host.
- **Private IPv4 and IPv6 addresses are retained**.
- The ordinary dynamic **public IPv4 address is released**.
- An associated **Elastic IP is retained**.

### What remains and what changes

| Layer | Stop / Start behavior |
|---|---|
| EC2 compute | Stops, then can start again |
| RAM | Lost |
| EBS | Remains attached; still billed |
| Private IPv4 | Retained |
| IPv6 | Retained |
| Dynamic public IPv4 | Released |
| Elastic IP | Retained |
| Physical host | Can change after Start |

### AWS-Onion meaning

~~~text
USER / ADMIN
      ↓
EC2 CONTROL PLANE
      ↓
COMPUTE STOPS
      ↓
RAM DISAPPEARS
      ↓
EBS PERSISTS
      ↓
NETWORK IDENTITY PARTLY PERSISTS
      ↓
PHYSICAL COMPUTE CAN BE REPLACED
~~~

This is one of the clearest examples of abstraction in AWS:

> **Logical EC2 identity ≠ permanent physical server.**

And it teaches another core rule:

> **Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle.**

---

## 3. HIBERNATING an EC2 instance

### What the lesson says

- Applies only to **supported AMIs**.
- Hibernation must be **enabled when the instance is launched**.
- Specific prerequisites apply.
- The contents of **RAM are saved to the EBS volume**.
- When the instance starts again:
  - the EBS root volume returns to its previous state,
  - RAM contents are reloaded,
  - previously running processes resume,
  - previously attached data volumes are reattached,
  - the instance retains its instance ID.

### The essential difference: Stop vs Hibernate

~~~text
STOP
RAM ───────────────→ LOST

HIBERNATE
RAM ──save──→ EBS
               │
               └──reload──→ RAM
                              │
                              └── processes resume
~~~

### AWS-Onion meaning

Hibernate creates a relationship between two normally different lifecycle domains:

~~~text
COMPUTE / RAM
      ↓
PERSIST TO
      ↓
EBS STORAGE
      ↓
RESTORE TO
      ↓
COMPUTE / RAM
~~~

> **Storage temporarily preserves volatile Compute state.**

---

## 4. REBOOTING an EC2 instance

### What the lesson says

- Equivalent to an **operating-system reboot**.
- The DNS name is retained.
- IPv4 and IPv6 addresses are retained.
- Billing is not affected.

### AWS-Onion meaning

Reboot is a comparatively shallow lifecycle operation:

~~~text
USER
 ↓
EC2 CONTROL
 ↓
GUEST OS REBOOT
 ↓
RUNNING AGAIN
~~~

The deeper persistent layers remain essentially intact.

> **Reboot mainly affects the OS/runtime layer, not the identity of the EC2 architecture around it.**

---

## 5. RETIRING an EC2 instance

### What the lesson says

An EC2 instance may be scheduled for retirement when AWS detects an **irreparable failure of the underlying hardware** hosting the instance.

At the scheduled retirement time, AWS stops or terminates the instance.

### AWS-Onion meaning

This is especially important because the problem originates **below the virtual machine**:

~~~text
EC2 / OS
   │
   ▼
AWS VIRTUALIZATION / PLATFORM
   │
   ▼
PHYSICAL HOST
   X
HARDWARE FAILURE
~~~

The application or guest operating system may not be the cause at all.

> **Retirement demonstrates the separation between the logical EC2 instance and the physical infrastructure that hosts it.**

---

## 6. TERMINATING an EC2 instance

### What the lesson says

- Termination means **deleting the EC2 instance**.
- A terminated instance **cannot be recovered**.
- By default, the **root EBS volume is deleted**.

### Stop is not Terminate

~~~text
STOP
EC2 can return
EBS persists
        ↓
START AGAIN

TERMINATE
EC2 deleted
Root EBS deleted by default
        ↓
NO RETURN
~~~

### AWS-Onion meaning

> **Stop preserves the logical machine for later use. Terminate ends the machine's lifecycle.**

This distinction is critical both architecturally and for the certification exam.

---

## 7. RECOVERING an EC2 instance

### What the lesson says

- CloudWatch can monitor **system status checks**.
- Recovery can be used when an instance becomes impaired because of **underlying hardware or platform issues**.
- The recovered instance is intended to be identical to the original instance.

### AWS-Onion meaning — the return path

Here the direction reverses.

The problem originates deep in the infrastructure and the evidence moves upward:

~~~text
UNDERLYING HARDWARE / PLATFORM
             │
             ↑
       SYSTEM STATUS
             ↑
         CLOUDWATCH
             ↑
 ADMINISTRATOR / AUTOMATION
~~~

Then a corrective decision creates a new downstream flow:

~~~text
ADMINISTRATOR / AUTOMATION
             ↓
         RECOVERY
             ↓
            EC2
             ↓
 HEALTHY INFRASTRUCTURE
~~~

This gives us the full operational AWS-Onion loop:

> **↓ Action goes downstream. ↑ Evidence comes upstream. A new decision starts another downstream action.**

---

## 8. One comparison table for the whole lesson

| Operation | RAM | EBS | IP / network identity | Physical host | Can instance return? | Core idea |
|---|---|---|---|---|---|---|
| **Reboot** | Runtime restarts | Retained | DNS, IPv4 and IPv6 retained | Running environment remains | Yes | OS-level restart |
| **Stop / Start** | Lost | Retained; chargeable | Private IPv4 + IPv6 retained; dynamic public IPv4 released; Elastic IP retained | Can change | Yes | Compute is separable from storage and host |
| **Hibernate / Resume** | Saved to EBS and restored | Retained | Instance identity retained under hibernation lifecycle | Runtime is reconstructed | Yes | Preserve volatile compute state |
| **Retire** | Depends on AWS stop/terminate action | Depends on resulting action | Depends on resulting action | Underlying host has irreparable problem | Depends on retirement action | Physical infrastructure can fail independently |
| **Terminate** | Lost | Root EBS deleted by default | Instance networking relationship ends | Relationship ends | No | Permanent deletion |
| **Recover** | Recovery follows EC2 recovery mechanism | Original storage relationship preserved | Instance identity is preserved | Recovery addresses hardware/platform impairment | Yes | Observability drives corrective action |

---

## 9. AWS-Onion layer map for the lifecycle

~~~text
#1 USER / ADMINISTRATOR / AUTOMATION
        │
        │ Start / Stop / Reboot /
        │ Hibernate / Terminate / Recover
        ▼
#2 AWS CONTROL / MANAGEMENT
        │
        │ EC2 API / Console / CLI
        ▼
#3 COMPUTE
        │
        │ EC2 state
        ▼
#4 OPERATING SYSTEM / MEMORY
        │
        │ processes / RAM
        ▼
#5 STORAGE
        │
        │ EBS root + data volumes
        ▼
#6 NETWORK
        │
        │ ENI / Private IP / IPv6 /
        │ Public IPv4 / Elastic IP
        ▼
#7 AWS UNDERLYING INFRASTRUCTURE
        │
        │ platform / physical host
        │
        ↑
#8 STATUS / OBSERVABILITY
        │
        │ status checks / CloudWatch
        ↑
#1 ADMINISTRATOR / AUTOMATION
~~~

The numbering remains causal: **#1 is whoever initiates the particular action being modeled**.

---

## 10. Exam memory anchors

~~~text
REBOOT
→ OS reboot
→ addresses retained
→ billing continues

STOP
→ EBS-backed
→ RAM lost
→ EBS remains + is chargeable
→ private IPv4 / IPv6 retained
→ dynamic public IPv4 released
→ Elastic IP retained
→ Start can use a different host

HIBERNATE
→ supported AMI
→ enabled at launch
→ RAM saved to EBS
→ RAM + processes restored

RETIRE
→ underlying hardware irreparable
→ AWS schedules stop or termination

TERMINATE
→ delete instance
→ cannot recover it
→ root EBS deleted by default

RECOVER
→ hardware / platform impairment
→ system status + CloudWatch
→ recover the EC2 instance
~~~

## 11. AWS-Onion master formula

~~~text
INTENT / ACTION
      ↓
CONTROL
      ↓
COMPUTE STATE
      ↓
OS / MEMORY
      ↓
STORAGE + NETWORK
      ↓
PHYSICAL INFRASTRUCTURE
      ↑
STATUS / METRICS / ERRORS
      ↑
OBSERVABILITY
      ↑
NEXT DECISION
~~~

> **AWS-Onion does not ask only “What state is the EC2 in?” It asks “Which layer changed, which layer persisted, and what evidence returned to the administrator?”**

Related:
- [AWS-Onion Downstream Model](../01-core/DOWNSTREAM-MODEL.md)
- [EC2 Networking & Connectivity](ec2-networking.md)
- [CloudWatch](../04-operations/cloudwatch.md)
