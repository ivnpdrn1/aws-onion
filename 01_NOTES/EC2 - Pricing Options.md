---
type: exam-note
domain: compute
service: EC2
tags: [aws, ec2, pricing, savings-plans, reserved-instances, spot, capacity, dedicated, certification]
updated: 2026-10-09
---

# EC2 — Pricing Options

> [!summary]
> **PRICE ≠ CAPACITY ≠ TENANCY ≠ BILLING**

> [!tip]
> **Resource lifecycle ≠ Pricing commitment lifecycle.**

## Horizontal Onion

```text
EC2 ECONOMIC / CAPACITY LAYER
│
├── FLEXIBILITY
│      └── On-Demand
├── COMMITMENT / DISCOUNT
│      ├── Savings Plans
│      │      ├── Compute
│      │      └── EC2 Instance
│      └── Reserved Instances
│             ├── Standard
│             └── Convertible
├── INTERRUPTIBLE
│      └── Spot
├── CAPACITY ASSURANCE
│      ├── Capacity Reservation
│      └── Zonal RI
└── PHYSICAL ISOLATION
       ├── Dedicated Instance
       └── Dedicated Host
```


## Course examples — identify the workload first

> [!important]
> These examples come from the class screenshots. Use them to recognize **workload characteristics → purchasing option**.

| Workload in the class | Think | Why |
|---|---|---|
| Developer, small project, several hours, cannot be interrupted | **On-Demand** | short-term + flexible + no Spot interruption |
| Steady-state, business-critical, continuous demand | **Reserved** | predictable continuous usage |
| Reporting app, 6 h/day, 4 days/week | **Scheduled Reserved** | predictable recurring schedule |
| Compute-intensive, cost-sensitive, distributed, interruption-tolerant | **Spot** | interruption accepted for lower cost |
| Security-sensitive, dedicated hardware, per-instance billing | **Dedicated Instance** | isolation |
| Database with per-socket licensing | **Dedicated Host** | socket/core/host visibility |

> [!warning]
> **Scheduled Reserved is historical/course-era material.** Preserve it for understanding the lesson, but do not select it for a new current AWS design.

### Six instant associations

```text
few hours + cannot interrupt       → ON-DEMAND
steady + continuous                → RESERVED
recurring fixed schedule           → SCHEDULED RESERVED [historical]
distributed + interruptible        → SPOT
security-sensitive + isolation     → DEDICATED INSTANCE
per-socket licensing               → DEDICATED HOST
```

### Exam habit

> **First identify the workload characteristic. Then choose the purchasing option.**

```text
WORKLOAD CLUE
     ↓
SPECIALTY BEING TESTED
     ↓
EC2 PURCHASING OPTION
```


## Exam triggers

| Trigger | Think |
|---|---|
| unpredictable / no commitment | On-Demand |
| $/hour commitment | Savings Plans |
| specific configuration commitment | Reserved Instance |
| steady predictable workload | Savings Plans / RI |
| interruptible / fault tolerant | Spot |
| capacity guarantee in AZ | Capacity Reservation / Zonal RI |
| physical isolation | Dedicated Instance |
| socket/core/host ID/BYOL | Dedicated Host |

## Course → current AWS corrections

> [!warning]
> **Spot Blocks are historical.** New customers lost access in 2021 and remaining support ended in 2022.

> [!warning]
> **RHEL hourly billing in the course is outdated.** RHEL changed to per-second billing in 2024.

> [!warning]
> **EC2 Fleet launches Spot + On-Demand capacity.** Reserved Instance and Savings Plans discounts can apply to matching On-Demand usage; RI is not a third Fleet launch target.

> [!note]
> The course says CloudWatch Events for Spot interruption. Current product terminology is **Amazon EventBridge**.

> [!note]
> AWS currently recommends **Savings Plans over EC2 Reserved Instances** for most commitment-based EC2 savings, while RIs remain important for certification and zonal capacity behavior.

## Memory block

```text
ON-DEMAND        = flexibility
SAVINGS PLAN     = commit $/hour
RESERVED         = configuration commitment
SPOT             = spare + interruptible
CAPACITY RES.    = capacity assurance
DEDICATED INST.  = isolation
DEDICATED HOST   = host-level control
```

## Three-axis exam method

```text
WHERE?
  ↓ Vertical layer
WHAT BUILDS IT?
  ↓ Horizontal option
WHAT HAPPENS NEXT?
  ↓ Functional cost/capacity effect
```

> **WHERE → WHAT INSIDE → WHAT NEXT**

## Exam scenario — Per-socket licenses and host placement (2026-10-15)

**Question:** Organization uses per-socket licensed software and needs control over EC2 placement on underlying physical hardware.

**Correct:** Dedicated Hosts. Host-level identification, visibility into physical sockets/cores and placement/affinity controls support eligible bring-your-own-license requirements (always verify vendor license rules).

| Option | What it solves | Why it is not the answer |
|---|---|---|
| Dedicated Hosts | Dedicated physical host and placement control, socket/core visibility | **Meets both explicit requirements** |
| Dedicated Instances | Single-tenant hardware isolation for EC2 | No equivalent host ID, physical socket/core visibility and host placement control |
| Reserved Instances | Billing discount; some zonal RI capacity reservation | Not a physical host allocation or socket licensing mechanism |
| Spot Instances | Discounted, interruptible spare EC2 capacity | Not hardware placement or host-based license control |

**Memory rule:** **Dedicated Instance = isolate my VM; Dedicated Host = identify and place my VM on a physical server.** Discount (RI), spare capacity (Spot), tenancy (Dedicated Instance) and host control (Dedicated Host) are *different decision axes*.

## Related
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[EC2 - Instance Lifecycle]]
- [[EC2 - Nitro System and Nitro Enclaves]]
- [[EC2 - Networking and Connectivity]]
