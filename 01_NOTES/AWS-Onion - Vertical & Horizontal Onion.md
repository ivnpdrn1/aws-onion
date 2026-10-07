---
type: concept
domain: architecture
tags: [aws, onion, vertical-onion, horizontal-onion, functional-onion, certification]
---

# AWS-Onion — Vertical & Horizontal Onion

> [!abstract] Core model
> **Vertical = BETWEEN layers. Horizontal = WITHIN a layer. Functional = THROUGH time.**

## Three-axis model

| Axis | Ask |
|---|---|
| **Vertical Onion** | What layer depends on what? |
| **Horizontal Onion** | What components build this layer's specialty? |
| **Functional Onion** | What changes when an action occurs? |

```text
AWS-ONION
   │
   ├── VERTICAL   → BETWEEN layers
   ├── HORIZONTAL → WITHIN a layer
   └── FUNCTIONAL → THROUGH time
```

## Vertical Onion

```text
#1 INITIATOR
   ↓
SERVICE / CONTROL
   ↓
ACCESS / NETWORK
   ↓
RESOURCE / EXECUTION
   ↓
PLATFORM / INFRASTRUCTURE
   ↑
STATUS / EVIDENCE
```

> [!tip]
> **↓ Downstream = intent/action. ↑ Upstream = evidence/result.**

## Horizontal Onion

```text
ONE LAYER
   │
   ├── Component A
   ├── Component B
   ├── Component C
   └── Component D
          ↓
     LAYER SPECIALTY
```

> [!tip]
> **A layer is a function assembled from specialized components.**

## Exam method

```text
WHERE?
  ↓
VERTICAL LAYER

WHAT BUILDS IT?
  ↓
HORIZONTAL COMPONENT

WHAT HAPPENS NEXT?
  ↓
FUNCTIONAL EFFECT
```

> **WHERE → WHAT INSIDE → WHAT NEXT**

## Related
- [[AWS-Onion - Mental Model]]
- [[AWS-Onion - Functional Onion]]
- [[EC2 - Nitro System and Nitro Enclaves]]
