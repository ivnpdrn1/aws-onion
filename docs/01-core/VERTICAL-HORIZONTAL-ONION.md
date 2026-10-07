# AWS-Onion — Vertical, Horizontal and Functional Axes

## Purpose

AWS-Onion is a relationship-first model. A single diagram is not enough to explain all AWS relationships, so the framework uses **three complementary axes**.

| Axis | Core question | Relationship type | Exam value |
|---|---|---|---|
| **Vertical Onion** | What layer depends on what other layer? | Inter-layer | Reconstruct architecture, path, dependency and troubleshooting order |
| **Horizontal Onion** | What components build the function of this layer? | Intra-layer | Understand why a layer can perform its specialty |
| **Functional Onion** | What changes when an action occurs? | Temporal / causal | Follow state, persistence, side effects and recovery |

> **Vertical = BETWEEN layers.**  
> **Horizontal = WITHIN a layer.**  
> **Functional = THROUGH time.**

These are complementary views, not competing definitions.

---

# 1. Structural Onion now has two directions

The previous **Structural Onion** remains the model for understanding what exists and how components relate.

It now has two structural views:

~~~text
STRUCTURAL ONION
      │
      ├── VERTICAL ONION
      │     relationships BETWEEN specialized layers
      │
      └── HORIZONTAL ONION
            components WITHIN one specialized layer
~~~

The **Functional Onion** remains the temporal view:

~~~text
FUNCTIONAL ONION
      │
      └── what happens NEXT when an action changes state
~~~

Mental rule:

> **Structure tells us WHERE and WHAT. Function tells us WHEN and WHAT CHANGES.**

---

# 2. AWS Vertical Onion

## Definition

The **AWS Vertical Onion** models the dependency and causal relationship from one specialized layer to another.

A layer should have a clear specialty. It consumes capabilities from lower/dependent layers and provides a capability to the layer above/next.

Canonical form:

~~~text
#1 INITIATOR / USER
        ↓
#2 SERVICE / CONTROL
        ↓
#3 ACCESS / NETWORK PATH
        ↓
#4 RESOURCE / EXECUTION
        ↓
#5 PLATFORM / INFRASTRUCTURE

        ↑
STATUS / METRICS / EVENTS / ERRORS
        ↑
OBSERVABILITY / OPERATOR
~~~

### Downstream

> **↓ Downstream = intent, requests, configuration and actions.**

### Upstream

> **↑ Upstream = status, metrics, events, errors and evidence.**

This is a conceptual dependency/causality model. It is not a claim that every AWS component is a physical packet hop.

---

# 3. AWS Horizontal Onion

## Definition

The **AWS Horizontal Onion** freezes one vertical layer and opens it to show the internal components that collectively create that layer's specialty.

Canonical form:

~~~text
VERTICAL LAYER: X
      │
      └── SPECIALTY / FUNCTION
             │
             ├── Component A
             ├── Component B
             ├── Component C
             └── Component D
                    ↓
             LAYER CAPABILITY
~~~

The Horizontal Onion answers:

1. What is this layer's specialty?
2. Which components produce that specialty?
3. What responsibility does each component own?
4. Which components are required versus optional?
5. Which components are transversal?
6. What does the completed layer provide to the next vertical layer?
7. Which component names are likely to appear as exam distractors?

Mental rule:

> **A layer is not just a box. It is a function assembled from specialized components.**

---

# 4. The relationship between both axes

The Vertical Onion explains **interdependence**.

The Horizontal Onion explains **composition**.

~~~text
                    HORIZONTAL VIEW
              ┌─────────────────────────┐
              │ COMPONENT A             │
              │ COMPONENT B             │
VERTICAL  →   │ COMPONENT C             │  → LAYER FUNCTION
LAYER         │ COMPONENT D             │
              └─────────────────────────┘
                        │
                        ▼
                  NEXT VERTICAL LAYER
~~~

Therefore:

> **Vertical Onion = how AWS functions connect.**  
> **Horizontal Onion = how each AWS function is built.**

---

# 5. Third axis — Functional Onion

A static architecture can still behave differently when an action occurs.

The Functional Onion adds the temporal dimension:

~~~text
STATE A
   ↓ action
STATE B
   ↓ effects
DEPENDENT COMPONENTS CHANGE / PERSIST / RELEASE
   ↓ evidence
NEXT STATE / DECISION
~~~

Examples:

- EC2 Running → Stop → Stopped
- RAM → LOSE
- EBS → KEEP
- dynamic public IPv4 → RELEASE
- private IPv4 → KEEP
- evidence → returns upstream

Therefore the complete reasoning model is:

~~~text
                AWS-ONION
                    │
        ┌───────────┼───────────┐
        │           │           │
     VERTICAL    HORIZONTAL   FUNCTIONAL
        │           │           │
 BETWEEN layers  WITHIN layer  THROUGH time
        │           │           │
 dependency      composition    change/state
~~~

---

# 6. Exam reconstruction method

When an AWS exam question describes an unfamiliar scenario, reconstruct it in three passes.

## Pass 1 — Vertical

Ask:

> **Where in the architecture is the problem or requirement?**

Examples:
- identity?
- network path?
- compute?
- storage?
- observability?
- underlying infrastructure?

## Pass 2 — Horizontal

Ask:

> **Which component inside that layer owns the required function?**

Examples:
- route table versus Security Group
- ENI versus ENA
- Nitro Card versus Nitro Hypervisor
- IAM policy versus Role session

## Pass 3 — Functional

Ask:

> **What changes if the requested action occurs?**

Examples:
- persists?
- is released?
- is recreated?
- is restored?
- emits evidence?
- changes underlying placement?

This converts memorization into reconstruction.

---

# 7. Master exam formula

~~~text
WHERE?
  ↓
VERTICAL LAYER

WHAT BUILDS IT?
  ↓
HORIZONTAL COMPONENT

WHAT HAPPENS NEXT?
  ↓
FUNCTIONAL EFFECT
~~~

> **WHERE → WHAT INSIDE → WHAT NEXT**

This is the default AWS-Onion study sequence.

---

# 8. Example — Networking

## Vertical

~~~text
Internet
   ↓
IGW
   ↓
Route / Subnet
   ↓
NACL
   ↓
Security Group
   ↓
ENI
   ↓
EC2
   ↓
Application
~~~

## Horizontal — Network/Connectivity layer

~~~text
NETWORK / CONNECTIVITY
        │
        ├── VPC
        ├── CIDR
        ├── Subnet
        ├── Route Table
        ├── IGW / service access path
        ├── NACL
        ├── Security Group
        ├── ENI
        └── IP addressing
                ↓
        REACHABILITY + CONTROL
~~~

## Functional

~~~text
configuration/action
       ↓
routing / filtering / addressing changes
       ↓
reachability result
       ↑
logs / errors / status
~~~

---

# 9. Example — Nitro

Nitro is a strong Horizontal Onion example because AWS separates infrastructure functions into specialized modules.

## Vertical view

~~~text
APPLICATION / WORKLOAD
        ↓
EC2 INSTANCE
        ↓
NITRO SYSTEM
        ↓
PHYSICAL AWS INFRASTRUCTURE
~~~

For virtualized instances, the Nitro Hypervisor participates in the virtualization layer. For bare-metal instances, the operating system accesses the physical infrastructure without a traditional virtualization layer between them.

## Horizontal view of the Nitro layer

~~~text
NITRO SYSTEM
     │
     ├── Nitro Hypervisor
     ├── Nitro Cards for VPC
     ├── Nitro Cards for EBS
     ├── Nitro Cards for instance storage
     ├── Nitro Controller
     ├── Nitro Security Chip
     └── Nitro Enclaves
            ↓
 PERFORMANCE + SECURITY + SPECIALIZATION
~~~

This is the key pattern:

> **Separate responsibilities → specialized components → lower virtualization overhead + stronger isolation/security.**

See: [AWS Nitro System and Nitro Enclaves](../02-compute-networking/nitro-system-enclaves.md).

---

# 10. Memory anchors

> **Vertical = BETWEEN layers.**  
> **Horizontal = WITHIN a layer.**  
> **Functional = THROUGH time.**

> **Downstream = intent/action. Upstream = evidence/result.**

> **Each layer has a specialty. Its horizontal components exist to build that specialty.**

> **Exam method: WHERE → WHAT INSIDE → WHAT NEXT.**

Related:
- [Core AWS-Onion Mental Model](aws-onion-core.md)
- [AWS-Onion Downstream Model](DOWNSTREAM-MODEL.md)
- [AWS-Onion Functional Model](FUNCTIONAL-ONION.md)
