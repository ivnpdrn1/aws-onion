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

## Related
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[EC2 - Instance Lifecycle]]
- [[EC2 - Nitro System and Nitro Enclaves]]
- [[EC2 - Networking and Connectivity]]
