# EC2 Instance Lifecycle through AWS-Onion

## Purpose

The EC2 instance lifecycle is a practical demonstration of the AWS-Onion philosophy:

> One user action can cross several architectural layers, while each layer preserves, changes or releases different parts of the system.

The lifecycle should therefore be learned as **relationships between Compute, Operating System state, Storage, Network identity, Physical infrastructure and Observability**, not only as a list of EC2 states.

## Canonical EC2 states

~~~text
AMI
 ↓
PENDING
 ↓
RUNNING
 ├── Reboot ───────────────→ RUNNING
 ├── Stop ─→ STOPPING ─────→ STOPPED ─→ Start ─→ PENDING ─→ RUNNING
 ├── Hibernate ─→ STOPPING → STOPPED ─→ Start / resume
 └── Terminate → SHUTTING-DOWN → TERMINATED
~~~

A terminated instance cannot be recovered.

## AWS-Onion lifecycle flow

~~~text
#1 USER / ADMINISTRATOR / AUTOMATION
        │
        │ Start / Stop / Reboot / Hibernate / Terminate
        ▼
#2 AWS CONTROL / MANAGEMENT
        │
        │ EC2 API / Console / CLI
        ▼
#3 COMPUTE
        │
        │ EC2 instance state transition
        ▼
#4 GUEST OPERATING SYSTEM / MEMORY
        │
        ├── processes
        └── RAM
        ▼
#5 STORAGE
        │
        └── EBS root / data volumes
        ▼
#6 NETWORK
        │
        ├── ENI
        ├── Private IPv4
        ├── IPv6
        ├── Public IPv4
        └── Elastic IP
        ▼
#7 AWS UNDERLYING INFRASTRUCTURE
        │
        └── physical host / platform
        ↑
#8 STATUS / OBSERVABILITY
        │
        ├── system status
        ├── instance status
        └── CloudWatch visibility
        ↑
      ADMINISTRATOR
~~~

The numbering represents causal reasoning, not independent physical appliances.

## What each lifecycle action teaches

| Action | Compute / OS | RAM | EBS | Network identity | Underlying host | AWS-Onion lesson |
|---|---|---|---|---|---|---|
| Reboot | OS reboots; instance returns to running | Reinitialized as part of reboot | Remains | DNS and IPv4/IPv6 addresses are retained | Instance remains associated with its running environment | A shallow state change can leave lower architectural layers intact |
| Stop | EC2 compute stops | Lost | EBS remains attached and billable | Private IPv4 and IPv6 retained; ordinary public IPv4 released; Elastic IP retained | A later Start can place the instance on a different host | Compute and persistent storage are separate lifecycle domains |
| Start after Stop | Instance goes Pending → Running | New runtime memory | Existing EBS is reused | Persistent network identity is restored according to the retained resources | Can execute on a different physical host | Logical instance identity is abstracted from a specific physical server |
| Hibernate | Compute stops after preserving memory state | Saved to EBS | Persists saved memory plus volumes | Instance identity and attached data volumes are retained | Resume reconstructs prior runtime state | Storage can preserve part of compute state |
| Terminate | Instance is deleted | Lost | Root EBS is deleted by default | Instance networking is released with the instance | Compute relationship ends | Terminate is destructive, unlike Stop |
| Recover | Used when impairment comes from underlying hardware/platform issues | Runtime is recovered according to the recovery mechanism | Original instance storage relationship is preserved | Instance identity is preserved | AWS moves recovery away from the impaired infrastructure | Observability can trigger corrective action across lower layers |

## Stop versus Terminate

~~~text
STOP
  ↓
Compute paused / released
  ↓
Persistent EBS remains
  ↓
Instance can be started again

TERMINATE
  ↓
Instance deleted
  ↓
Root EBS deleted by default
  ↓
No return to Running
~~~

Mental anchor:

> **STOP preserves the logical machine for later use. TERMINATE ends its lifecycle.**

## Stop reveals abstraction

A Stop followed by Start may move the EC2 instance to a different physical host.

~~~text
BEFORE
Logical EC2
    ↓
Physical Host A

STOP / START

AFTER
Logical EC2
    ↓
Physical Host B
~~~

This is one of the clearest AWS-Onion examples of abstraction:

> **Logical compute identity is not the same thing as a permanent physical server.**

## Storage is a separate layer

For an EBS-backed instance:

~~~text
EC2 COMPUTE
     │
     │ attached to
     ▼
EBS STORAGE
~~~

Stopping compute does not mean deleting persistent storage.

This reinforces:

> **Compute lifecycle ≠ Storage lifecycle.**

EBS storage can continue to incur charges while the EC2 instance itself is stopped.

## Network is a separate layer

A Stop/Start cycle demonstrates that network attributes have their own persistence rules.

~~~text
EC2
 ↓
ENI
 ├── Private IPv4   → retained
 ├── IPv6           → retained
 ├── Public IPv4    → ordinary dynamic address released
 └── Elastic IP     → retained
~~~

Mental anchor:

> **The EC2 instance is not the address. It uses networking resources and identities supplied by another layer.**

## Hibernate crosses Compute and Storage

Hibernate preserves RAM contents by saving them to the EBS root volume. Hibernation must be enabled at launch and is available only when its prerequisites are satisfied.

~~~text
RUNNING RAM
    ↓
SAVE TO EBS
    ↓
STOPPED
    ↓
START
    ↓
RELOAD RAM
    ↓
RESUME PROCESSES
~~~

This is a useful Onion relationship:

> **Storage can preserve volatile compute state so that execution can later resume.**

## Underlying infrastructure and recovery

System-level impairment can originate below the EC2 guest itself, at the underlying AWS hardware/platform layer.

~~~text
UNDERLYING HOST / PLATFORM ISSUE
          ↑
   SYSTEM STATUS SIGNAL
          ↑
      CLOUDWATCH
          ↑
 ADMINISTRATOR / AUTOMATION
~~~

A corrective action can then flow downstream again:

~~~text
ADMINISTRATOR / AUTOMATION
          ↓
       RECOVERY
          ↓
         EC2
          ↓
HEALTHY UNDERLYING INFRASTRUCTURE
~~~

This produces the AWS-Onion operational loop:

> **Action flows down. Evidence flows up. The next decision starts a new downstream flow.**

## Core certification associations

Remember the relationships rather than isolated facts:

~~~text
Reboot    → OS-level restart; addresses retained
Stop      → EBS-backed; RAM lost; EBS persists; compute charge stops
Start     → Pending → Running; can use a different physical host
Hibernate → RAM saved to EBS; resume prior processes
Terminate → irreversible; root EBS deleted by default
Status    → System = underlying AWS infrastructure
             Instance = guest instance / OS
Recover   → impairment caused by hardware/platform layer
~~~

## AWS-Onion master formula

~~~text
INTENT
  ↓
MANAGEMENT / CONTROL
  ↓
COMPUTE STATE
  ↓
OS / MEMORY
  ↓
STORAGE + NETWORK + INFRASTRUCTURE EFFECTS
  ↑
STATUS / METRICS / EVENTS
  ↑
OBSERVABILITY
  ↑
NEXT HUMAN OR AUTOMATED DECISION
~~~

Related:
- [AWS-Onion Downstream Model](../01-core/DOWNSTREAM-MODEL.md)
- [EC2 Networking & Connectivity](ec2-networking.md)
- [CloudWatch](../04-operations/cloudwatch.md)
