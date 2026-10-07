---
type: concept
domain: architecture
tags: [aws, onion, functional-onion, lifecycle, causality, persistence]
---

# AWS-Onion — Functional Onion

## Two complementary views

| Structural Onion | Functional Onion |
|---|---|
| What components/layers exist? | What happens after an action? |
| Relationship in architecture | Relationship through time |
| Dependency / placement | Transition / persistence |
| “Where is it?” | “What happens next?” |

> **Structural Onion = WHAT is connected.**  
> **Functional Onion = WHAT HAPPENS NEXT.**

## Functional sequence

~~~text
INITIATOR
   ↓
ACTION
   ↓
PRIMARY RESOURCE
   ↓
RUNTIME / RAM
   ↓
PERSISTENT STORAGE
   ↓
NETWORK IDENTITY
   ↓
PHYSICAL / PLATFORM
   ↑
STATUS / EVIDENCE
   ↑
NEXT DECISION
~~~

## Persistence labels

| Label | Meaning |
|---|---|
| KEEP | remains |
| LOSE | disappears |
| SAVE | persist elsewhere |
| RESTORE | load saved state |
| RELEASE | relinquish assignment |
| DELETE | end lifecycle |
| REATTACH | reconnect existing resource |
| MAY CHANGE | logical identity remains while placement may differ |
| NOT SPECIFIED | lesson does not define it |

## EC2 lifecycle anchor

> **EC2 is the primary block. RAM, EBS, network identity, physical host and observability are dependent/accessory blocks whose behavior must be checked for each lifecycle function.**

## Study formula

> **TRIGGER → TARGET → CHANGE → PERSIST → RELEASE → EVIDENCE → NEXT STEP**

## Related
- [[EC2 - Instance Lifecycle]]
- [[AWS-Onion - Mental Model]]
- [[CloudWatch - Mental Model]]
