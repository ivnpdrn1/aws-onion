# Amazon EC2 Pricing Options through AWS-Onion

## Purpose

This note preserves two distinct layers of knowledge:

1. **COURSE LESSON** — what the supplied EC2 Pricing Options lesson teaches and what may still appear in certification training material.
2. **CURRENT AWS (verified 2026)** — what current AWS documentation says today.

> **Never silently replace an exam lesson with current operations guidance. Preserve both and label the difference.**

The goal is to understand EC2 pricing as architecture, not as a list of prices.

---

# 1. Core mental model

An EC2 design asks at least two different questions:

~~~text
WHAT compute do I need?
        ↓
instance family / size / CPU / RAM / OS

HOW do I acquire and pay for it?
        ↓
EC2 purchasing / capacity / tenancy model
~~~

Core rule:

> **EC2 instance type = WHAT compute.**  
> **EC2 purchasing option = HOW that compute is acquired/paid for.**

A second rule is even more important:

> **PRICE ≠ CAPACITY ≠ TENANCY ≠ BILLING.**

These are related but independent dimensions.

---

# 2. AWS-Onion three-axis model

## Vertical Onion — where the decision sits

~~~text
#1 BUSINESS / APPLICATION REQUIREMENT
             ↓
#2 WORKLOAD CHARACTERISTICS
             │
             ├── predictable?
             ├── short-lived?
             ├── interruptible?
             ├── capacity-critical?
             ├── dedicated hardware?
             └── license-bound?
             ↓
#3 PURCHASING / CAPACITY / TENANCY MODEL
             │
             ├── On-Demand
             ├── Savings Plans
             ├── Reserved Instances
             ├── Spot
             ├── Capacity Reservations
             ├── Dedicated Instances
             └── Dedicated Hosts
             ↓
#4 EC2 INSTANCE / FLEET
             ↓
#5 NITRO SYSTEM
             ↓
#6 AWS PHYSICAL INFRASTRUCTURE
~~~

Upstream evidence:

~~~text
INFRASTRUCTURE / EC2 BEHAVIOR
             ↑
usage / interruptions / utilization
             ↑
CloudWatch / EventBridge / Billing / Cost Explorer
             ↑
operator / automation
             ↑
NEXT COST / CAPACITY DECISION
~~~

> **↓ Downstream = business intent becomes infrastructure.**  
> **↑ Upstream = utilization, events and cost become the next decision.**

---

# 3. Horizontal Onion — what builds the EC2 economic/capacity layer

~~~text
EC2 ECONOMIC / CAPACITY LAYER
│
├── FLEXIBILITY
│      └── On-Demand
│
├── COMMITMENT / DISCOUNT
│      ├── Savings Plans
│      │      ├── Compute Savings Plans
│      │      └── EC2 Instance Savings Plans
│      │
│      └── Reserved Instances
│             ├── Standard RI
│             └── Convertible RI
│
├── INTERRUPTIBLE / SPARE CAPACITY
│      ├── Spot Instances
│      ├── Spot Fleet
│      └── EC2 Fleet (Spot + On-Demand launch capacity)
│
├── CAPACITY ASSURANCE
│      ├── On-Demand Capacity Reservations
│      ├── Zonal Reserved Instances
│      └── Capacity Blocks (specialized use cases)
│
├── PHYSICAL ISOLATION / TENANCY
│      ├── Dedicated Instances
│      └── Dedicated Hosts
│
└── BILLING / METERING
       ├── EC2 compute
       └── EBS storage
~~~

Mental rule:

> **One EC2 workload can combine several of these dimensions at once.**

Example:

~~~text
Savings Plan
      +
On-Demand Capacity Reservation
      +
EC2 instance
~~~

The Savings Plan solves **cost** while the Capacity Reservation solves **capacity assurance**.

---

# 4. Course lesson — preserved concepts

The supplied lesson teaches these associations:

| Lesson concept | Memory association |
|---|---|
| **On-Demand** | no discount, no commitment; dev/test, short-term, unpredictable |
| **Reserved Instances** | 1 or 3 year commitment; steady/predictable workloads |
| **Spot** | unused capacity; very low cost; AWS can reclaim capacity |
| **Dedicated Instance** | host-level physical isolation; billed per instance |
| **Dedicated Host** | physical server dedicated to you; socket/core visibility; host affinity |
| **Savings Plans** | consistent compute usage commitment in USD/hour; 1 or 3 years |
| **Capacity Reservation** | reserve EC2 capacity in a specific AZ |
| **Spot Fleet** | maintain target capacity with Spot and On-Demand |
| **EC2 Fleet** | broader fleet orchestration |
| **Spot interruption** | two-minute warning; metadata + event-driven automation |
| **Spot Block** | 1–6 hour uninterrupted discounted Spot block (historical) |

The lesson also teaches:

> **Predictability → Commitment → Discount**

and:

> **Interruptibility → Spot savings**

and:

> **Host-level licensing/control → Dedicated Host**

---


# 4A. Course visual examples — workload → purchasing option

The class screenshots add a very useful exam-oriented layer: instead of starting with the AWS product name, start with the **workload pattern** and infer the purchasing option.

| Course example | Purchasing option shown | Why the example fits | Memory trigger |
|---|---|---|---|
| **Developer working on a small project for several hours; cannot be interrupted** | **On-Demand** | Short-lived work, no long-term commitment, and interruption is not acceptable | **short + flexible + uninterrupted → On-Demand** |
| **Steady-state, business-critical, line-of-business application; continuous demand** | **Reserved** | Continuous and predictable usage can justify a commitment in exchange for discount | **steady + predictable + continuous → Reserved** |
| **Reporting application, runs for 6 hours a day, 4 days per week** | **Scheduled Reserved** | The course uses this to illustrate predictable recurring usage at known times | **recurring schedule → Scheduled Reserved (historical)** |
| **Compute-intensive, cost-sensitive distributed computing; can withstand interruption** | **Spot Instances** | Distributed workload can tolerate loss of instances and prioritizes low cost | **distributed + cost-sensitive + interruptible → Spot** |
| **Security-sensitive application, requires dedicated hardware; per-instance billing** | **Dedicated Instances** | The requirement is physical isolation from other customers, while billing is tied to instances | **isolation + per-instance → Dedicated Instance** |
| **Database with per-socket licensing** | **Dedicated Hosts** | Host-level socket/core visibility supports server-bound licensing | **socket/core licensing → Dedicated Host** |

## Read the examples by function, not by product name

~~~text
WORKLOAD REQUIREMENT
       │
       ├── short-lived + cannot be interrupted
       │      └── On-Demand
       │
       ├── steady + continuous
       │      └── Reserved
       │
       ├── predictable recurring schedule
       │      └── Scheduled Reserved  [COURSE / HISTORICAL]
       │
       ├── distributed + interruption tolerant
       │      └── Spot
       │
       ├── security-sensitive + dedicated hardware
       │      └── Dedicated Instance
       │
       └── per-socket / server-bound licensing
              └── Dedicated Host
~~~

This creates a high-value certification habit:

> **Do not ask first: “Which EC2 pricing product do I remember?”**  
> **Ask first: “What characteristic of the workload is AWS testing?”**

## Horizontal Onion — examples mapped to specialties

| Horizontal specialty | Course example | AWS option |
|---|---|---|
| **Flexibility** | small developer project for several hours | On-Demand |
| **Predictable commitment** | continuous line-of-business application | Reserved |
| **Recurring schedule** | reporting app 6 hours/day, 4 days/week | Scheduled Reserved — historical |
| **Interruptible low-cost compute** | distributed compute that tolerates interruption | Spot |
| **Physical isolation** | security-sensitive workload | Dedicated Instance |
| **Host-level licensing/control** | database licensed per socket | Dedicated Host |

## Current-AWS warning for the course examples

The **Scheduled Reserved** example must remain in the repository because it is part of the course material and illustrates the intended exam relationship. However:

> **Scheduled Reserved Instances are a historical/course-era concept and should not be selected for a new current AWS architecture.**

For current design work, separate the underlying need from the historical product:

~~~text
"Runs on a predictable recurring schedule"
            ↓
first identify the CURRENT capacity / scheduling / commitment mechanism
rather than automatically choosing Scheduled Reserved
~~~

The other five examples remain useful as conceptual workload-to-option associations, subject to the current-AWS corrections elsewhere in this document.

## Six one-line exam anchors

~~~text
Developer / few hours / no interruption
→ ON-DEMAND

Steady-state / continuous business workload
→ RESERVED

Recurring fixed schedule
→ SCHEDULED RESERVED  [historical course concept]

Distributed / cost-sensitive / interruption tolerant
→ SPOT

Security-sensitive / dedicated hardware / per-instance
→ DEDICATED INSTANCE

Per-socket licensing / host visibility
→ DEDICATED HOST
~~~

---

# 5. On-Demand

Course and current AWS agree on the central concept:

> **ON-DEMAND = maximum flexibility, no long-term commitment.**

Use it when:

- demand is unpredictable,
- workloads are short-term,
- development/test is still evolving,
- commitment risk is greater than possible savings.

Exam trigger:

> **short-term + unpredictable + no commitment → On-Demand**

---

# 6. Savings Plans

Savings Plans exchange commitment for lower prices.

~~~text
CONSISTENT USAGE COMMITMENT
        ($ / hour)
             ↓
          1 or 3 years
             ↓
        discounted usage
~~~

## Compute Savings Plans

Current AWS association:

- EC2 across instance families,
- sizes,
- Regions,
- OS,
- tenancy,
- plus AWS Fargate and AWS Lambda.

Memory:

> **Compute Savings Plan = widest compute flexibility.**

## EC2 Instance Savings Plans

Current AWS association:

- commitment to an EC2 instance family,
- in a selected Region,
- flexible across size, OS and tenancy within that family/Region.

Memory:

> **EC2 Instance Savings Plan = deeper EC2-specific commitment.**

Exam distinction:

~~~text
Compute SP
   ↑ more flexible

EC2 Instance SP
   ↓ more specific
~~~

Important current AWS operational guidance:

> **AWS currently recommends Savings Plans over EC2 Reserved Instances for most commitment-based EC2 savings because Savings Plans are easier and more flexible.**

Reserved Instances still matter for exams and for specific capacity/reservation behaviors.

---

# 7. Reserved Instances

Reserved Instances are primarily a **billing discount**, not a separate physical EC2 machine.

~~~text
RUNNING EC2 USAGE
       ↓
matches RI attributes?
       ↓
      YES
       ↓
RI billing discount applies
~~~

Current AWS offers:

- **Standard RI**
- **Convertible RI**
- **1-year or 3-year terms**
- **All Upfront / Partial Upfront / No Upfront**

Memory:

> **More upfront payment → generally deeper discount.**

## Standard vs Convertible

~~~text
STANDARD RI
   → higher discount potential
   → less configuration flexibility

CONVERTIBLE RI
   → more configuration flexibility
   → can be exchanged
~~~

Current AWS maximum advertised savings are generally:

- Standard RI: **up to 72%**
- Convertible RI: **up to 66%**

The course's generic **up to 75%** figure should therefore be treated as a course-era value rather than the current general EC2 RI headline.

---

# 8. Regional RI vs Zonal RI

This distinction is essential.

~~~text
REGIONAL RI
    ↓
billing discount flexibility
across AZs in the Region
    ↓
NO capacity reservation

ZONAL RI
    ↓
discount
+
capacity reservation
in a specific AZ
~~~

Memory:

> **REGION = discount flexibility.**  
> **AZ = capacity reservation.**

Exam trap:

> **Reserved Instance does NOT automatically mean reserved capacity.**

Capacity is reserved when the RI is zonal / tied to a specific Availability Zone.

---

# 9. Reserved Instance modification — current AWS

The lesson describes Standard RI modification and Convertible RI exchange.

Current AWS documentation emphasizes that Standard or Convertible RIs can be modified for items such as:

- Availability Zone within the same Region,
- scope (Regional ↔ Zonal),
- instance size within the same family and generation when supported.

Instance-size modification has restrictions, including Linux/UNIX and default tenancy requirements for size flexibility.

Convertible RIs can additionally be **exchanged** for other Convertible RIs with different configurations, subject to current exchange rules.

Exam memory:

> **MODIFY = reshape compatible reservation attributes.**  
> **EXCHANGE = Convertible RI flexibility.**

---

# 10. Capacity Reservations

Capacity Reservations solve a different problem from Savings Plans:

~~~text
Savings Plan
     ↓
COST OPTIMIZATION

Capacity Reservation
     ↓
CAPACITY ASSURANCE
~~~

Current On-Demand Capacity Reservations can hold capacity in a **specific Availability Zone**.

Important modern nuance:

- **Immediate-use Capacity Reservations** can have no term commitment and can be canceled when no longer needed.
- **Future-dated Capacity Reservations** can include a commitment duration and cancellation charges may apply during that commitment.
- Current EC2 also includes other capacity-reservation mechanisms, including **Capacity Blocks** for specialized GPU/ML workloads.

Therefore the course sentence:

> "no term commitment / cancel any time"

is correct for the classic immediate-use model, but is **not a complete description of all current Capacity Reservation variants**.

Cost relationship:

> **Unused reserved capacity can still cost money.**

A Savings Plan or eligible Regional RI can provide a billing discount while the Capacity Reservation separately provides capacity assurance.

---

# 11. Spot Instances

Core relationship remains current:

~~~text
AWS SPARE CAPACITY
        ↓
      SPOT
        ↓
lower price
        ↕
interruption risk
~~~

Memory:

> **SPOT = cheap because interruptible.**

Best suited to:

- fault-tolerant workloads,
- batch,
- analytics,
- CI/CD workers,
- stateless/distributed compute,
- workloads that can checkpoint or retry.

Architecture question:

> **Can the workload survive losing this instance?**

If the answer is no, Spot should not be the sole capacity strategy.

---

# 12. Spot interruption — current operational model

The lesson says AWS provides a two-minute warning through instance metadata and CloudWatch Events.

Current terminology:

~~~text
AWS needs Spot capacity
        ↓
Spot interruption warning
        ↓
EventBridge event
+
Instance Metadata (IMDS)
        ↓
automation
        ↓
checkpoint / drain / redirect / replace
~~~

Current AWS still provides a two-minute notice before stop or termination on a best-effort basis. For hibernation, hibernation begins immediately rather than providing two minutes of preparation.

AWS also provides **EC2 instance rebalance recommendations**, which can signal elevated interruption risk before the interruption notice.

Historical terminology note:

> **CloudWatch Events → Amazon EventBridge**

The lesson's CloudWatch Events wording reflects the older product name.

AWS-Onion relationship:

> **Infrastructure condition ↑ Event/Evidence → Automation ↓ Corrective action**

This is a canonical upstream/downstream loop.

---

# 13. Spot Fleet and EC2 Fleet — corrected current relationship

## Spot Fleet

A Spot Fleet can maintain a desired target capacity using Spot and, depending on configuration, On-Demand capacity.

Mental model:

~~~text
TARGET CAPACITY
      ↓
SPOT FLEET
      ├── Spot
      └── On-Demand
~~~

## EC2 Fleet

Current AWS documentation defines EC2 Fleet launch capacity as:

~~~text
EC2 FLEET
   ├── Spot
   └── On-Demand
~~~

A current EC2 Fleet does **not** launch "Reserved Instances" as a third fleet purchase target.

Instead:

> **Reserved Instance and Savings Plans discounts can apply to matching On-Demand usage launched by the fleet.**

This corrects an important course-era simplification.

EC2 Fleet can also use On-Demand Capacity Reservations and Capacity Blocks where applicable.

Exam/operations memory:

> **Fleet chooses/maintains capacity. RI/Savings Plans can discount matching On-Demand capacity.**

---

# 14. Spot Blocks — historical, not current

The lesson teaches **Spot Blocks** (Spot Instances with a defined uninterrupted duration of 1–6 hours).

This must be preserved as historical course content, but **must not be used as a current AWS design recommendation**.

AWS states:

- unavailable to new customers since **July 1, 2021**,
- support for previous users ended **December 31, 2022**.

Therefore:

> **SPOT BLOCK = historical exam/course concept, not a current EC2 purchasing option.**

If a current architecture requires reserved future GPU capacity, investigate current **Capacity Blocks**, which are a different product/concept.

---

# 15. Dedicated Instances vs Dedicated Hosts

## Dedicated Instances

~~~text
DEDICATED INSTANCE
      ↓
single-tenant hardware
      ↓
AWS controls host placement
~~~

Primary association:

> **physical isolation from other AWS customers**

The lesson's "per instance" mental model is useful, but current pricing also includes a **Dedicated per-Region fee** in addition to instance usage pricing.

## Dedicated Hosts

~~~text
DEDICATED HOST
      ↓
physical server allocated for your use
      ↓
host visibility + placement control
~~~

Important capabilities:

- host ID,
- socket/core visibility,
- host affinity,
- targeted placement,
- BYOL/server-bound licensing.

Memory:

> **Dedicated Instance = isolation.**  
> **Dedicated Host = isolation + host-level visibility/control.**

---

# 16. Host affinity and resilience

A Dedicated Host can establish affinity between an EC2 instance and a host.

~~~text
INSTANCE A → HOST 1
INSTANCE B → HOST 2
~~~

This can help an application-level or OS-level cluster deliberately avoid correlated hardware failure.

This connects tenancy with resilience:

> **Placement decision → failure-domain decision.**

---

# 17. Billing granularity — course vs current AWS

The course states:

- Amazon Linux / Windows / Ubuntu → per second with 1-minute minimum.
- RHEL / SUSE → hourly.
- EBS → per second with 1-minute minimum.

Current AWS has changed part of this.

## Current EC2 billing headline

Per-second billing with a 60-second minimum is currently available for:

- Amazon Linux,
- Windows,
- Red Hat Enterprise Linux,
- Ubuntu,
- Ubuntu Pro,

across applicable purchase options.

**RHEL changed to per-second billing in April 2024.**

Current On-Demand pricing documentation states that **SUSE Linux Enterprise Server (SLES)** is billed as a full hour.

EBS provisioned storage also uses per-second billing with a 60-second minimum.

Therefore:

> **The lesson's RHEL hourly-billing statement is outdated.**

Mental rule:

> **Purchasing model ≠ billing granularity.**

---

# 18. Functional Onion — lifecycle and cost are independent

An EC2 state transition can change compute charges without ending commitments.

~~~text
EC2 RUNNING
    ↓ STOP
EC2 STOPPED
    │
    ├── instance compute usage → stops
    ├── EBS storage            → persists / can keep charging
    ├── Savings Plan term      → continues
    └── RI term                → continues
~~~

Core rule:

> **Resource lifecycle ≠ Pricing commitment lifecycle.**

Extend the existing AWS-Onion lifecycle rule:

~~~text
Compute lifecycle
≠ RAM lifecycle
≠ Storage lifecycle
≠ Network-address lifecycle
≠ Physical-host lifecycle
≠ Pricing-commitment lifecycle
~~~

This is high-value exam reasoning.

---

# 19. Current decision tree

~~~text
Is the workload unpredictable / short-term?
        │
       YES
        ↓
    ON-DEMAND

Is usage steady and predictable?
        │
       YES
        ↓
SAVINGS PLAN (current default preference)
or RI when RI-specific behavior is required

Can the workload tolerate interruption?
        │
       YES
        ↓
      SPOT

Must capacity be guaranteed in a specific AZ?
        │
       YES
        ↓
CAPACITY RESERVATION
or ZONAL RI

Need single-tenant physical isolation?
        │
       YES
        ↓
DEDICATED INSTANCE

Need host ID / socket / core / affinity / BYOL?
        │
       YES
        ↓
DEDICATED HOST
~~~

---

# 20. Exam trigger table

| Phrase in question | Think |
|---|---|
| no commitment / unpredictable | **On-Demand** |
| steady predictable usage | **Savings Plans / RI** |
| $/hour compute commitment | **Savings Plans** |
| widest EC2/Fargate/Lambda flexibility | **Compute Savings Plan** |
| EC2 family + Region commitment | **EC2 Instance Savings Plan** |
| specific instance configuration commitment | **Reserved Instance** |
| specific AZ + RI capacity | **Zonal RI** |
| discount across AZs in a Region | **Regional RI** |
| lowest cost + fault tolerant | **Spot** |
| two-minute interruption notice | **Spot + EventBridge/metadata** |
| target capacity with Spot + On-Demand | **Spot Fleet / EC2 Fleet** |
| guarantee On-Demand capacity in an AZ | **Capacity Reservation** |
| physical isolation | **Dedicated Instance** |
| sockets / cores / host ID / BYOL | **Dedicated Host** |
| 1–6 hour Spot Block | **Historical/deprecated concept** |

---

# 21. Operational architecture pattern

A real workload often mixes models rather than selecting exactly one:

~~~text
APPLICATION CAPACITY
       │
       ├── BASELINE
       │      ↓
       │ Savings Plans / eligible commitment
       │
       ├── VARIABLE BURST
       │      ↓
       │ On-Demand
       │
       ├── INTERRUPTIBLE WORK
       │      ↓
       │ Spot
       │
       └── MISSION-CRITICAL AZ CAPACITY
              ↓
         Capacity Reservation
~~~

Then observe:

~~~text
CloudWatch / EventBridge / Cost Explorer
                 ↑
       utilization + events + cost
                 ↑
            architecture
                 ↓
          next optimization
~~~

This is the closed AWS-Onion economics loop.

---

# 22. Master memory block

~~~text
ON-DEMAND
= flexibility

SAVINGS PLAN
= commit $/hour

RESERVED INSTANCE
= configuration-based billing commitment

SPOT
= spare capacity / interruptible

CAPACITY RESERVATION
= capacity assurance

DEDICATED INSTANCE
= single-tenant isolation

DEDICATED HOST
= physical host + host-level control
~~~

Then add:

> **PRICE ≠ CAPACITY ≠ TENANCY ≠ BILLING**

and:

> **Resource lifecycle ≠ Pricing commitment lifecycle**

and the global AWS-Onion method:

> **WHERE → WHAT INSIDE → WHAT NEXT**

---

# 23. Course-to-current reconciliation table

| Course statement | Current AWS 2026 status |
|---|---|
| RI discount up to 75% | General current EC2 RI headline is up to **72%**; Convertible up to **66%** |
| RHEL billed hourly | **Outdated** — RHEL moved to per-second billing in 2024 |
| SUSE billed hourly | Still consistent with current On-Demand documentation |
| CloudWatch Events for Spot warning | Current name is **Amazon EventBridge** |
| EC2 Fleet includes Spot, On-Demand and Reserved as launch targets | **Correct current model:** EC2 Fleet launches Spot + On-Demand; RI/Savings discounts may apply to matching On-Demand |
| Spot Block 1–6 hours | **Historical** — feature support ended in 2022 |
| Capacity Reservation has no term commitment | Correct for immediate-use ODCR; current future-dated CRs can have commitment duration |
| Compute + EC2 Savings Plans | Still the two Savings Plan types relevant to EC2, although AWS now has other Savings Plan families for other services |
| RIs are key commitment option | Still valid for exam and supported, but AWS now generally recommends **Savings Plans over RIs** for most EC2 savings use cases |

---

# 24. Official AWS sources — current verification

- EC2 billing and purchasing options: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html
- 2026 EC2 purchasing decision guide: https://docs.aws.amazon.com/decision-guides/latest/decision-guides/ec2-purchasing-options-aws-how-to-choose.html
- EC2 Reserved Instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html
- RI modification: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ri-modifying.html
- Savings Plans types: https://docs.aws.amazon.com/savingsplans/latest/userguide/plan-types.html
- Capacity Reservations: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-capacity-reservations.html
- Spot Instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html
- Spot interruption notices: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-instance-termination-notices.html
- EC2 Fleet / Spot Fleet: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Fleets.html
- Dedicated Host affinity: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-understanding.html
- Current EC2 pricing: https://aws.amazon.com/ec2/pricing/
- Current EC2 Reserved Instance pricing: https://aws.amazon.com/ec2/pricing/reserved-instances/
- Dedicated Instance pricing: https://aws.amazon.com/ec2/pricing/dedicated-instances/
