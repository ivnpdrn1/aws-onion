---
type: dashboard
project: AWS-Onion
status: active
updated: 2026-10-09
tags: [aws-onion, aws, certification, architecture, developer, ai]
---

# AWS-Onion — Dashboard

> [!abstract] Purpose
> Build a durable AWS mental model through **relationships between layers**, prepare for AWS certifications, and evolve the learning system into a future book.

## Current thesis

> **Do not memorize AWS as isolated services. Understand what each layer depends on, what it controls, and what it provides to the next layer.**

## Certification paths

| Track | Current purpose |
|---|---|
| Solutions Architect | Architecture, networking, compute, storage, security, resilience |
| Developer | Application services, APIs, events, serverless, IAM, deployment |
| AI | Bedrock, AI services, agents, data, security, responsible AI |

## Core Onion

```text
ORIGIN
  ↓
GATEWAY
  ↓
PATH / ROUTING
  ↓
SUBNET CONTROL
  ↓
RESOURCE CONTROL
  ↓
NETWORK INTERFACE
  ↓
COMPUTE
  ↓
APPLICATION
```

## Navigation

### Core
- [[AWS-Onion - Mental Model]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[AWS-Onion - Functional Onion]]
- [[AWS-Onion - Exam Map]]
- [[AWS-Onion - Glossary]]

### Networking & Compute
- [[EC2 - Networking and Connectivity]]
- [[EC2 - Instance Lifecycle]]
- [[EC2 - Nitro System and Nitro Enclaves]]
- [[EC2 - Pricing Options]]

### IAM
- [[IAM - Groups Roles and STS]]

### Operations
- [[Systems Manager - Mental Model]]
- [[CloudWatch - Mental Model]]

### Book
- [[AWS-Onion - Book Vision]]

## Current memory anchors

> [!tip] Exam anchors
> **Route ≠ Address ≠ Permission**  
> **Price ≠ Capacity ≠ Tenancy ≠ Billing**  
> **ENI connects. ENA accelerates. EFA coordinates clusters.**  
> **GROUP = same identity + permissions through membership.**  
> **ROLE = temporary Role session + temporary credentials.**  
> **Agent communicates. Role authorizes. Network provides the path. SSM manages.**  
> **↓ Actions descend. ↑ Evidence returns.**  
> **Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.**  
> **Resource lifecycle ≠ Pricing commitment lifecycle.**  
> **Vertical = BETWEEN layers. Horizontal = WITHIN a layer. Functional = THROUGH time.**  
> **Exam reconstruction: WHERE → WHAT INSIDE → WHAT NEXT.**  
> **Nitro = performance through specialization. Enclave = security through isolation.**  
> **On-Demand = flexibility. Savings Plan = commit $/hour. Spot = interruptible spare capacity.**

## Method for every new AWS concept

1. What problem does it solve?
2. **Vertical:** where does it sit and what layers surround it?
3. **Horizontal:** which components build this layer's specialty?
4. What does it depend on?
5. What does it provide to the next layer?
6. Which principal needs permission?
7. What network path is required?
8. **Functional:** what changes when an action occurs?
9. What is the certification-exam association?
10. How does it connect with concepts already learned?
11. **Currency check:** is the course concept still current AWS behavior?

## Repository relationship

GitHub is the canonical source and backup. This vault-oriented structure is designed so the repository can also be opened directly as an Obsidian vault.

- Repository: `ivnpdrn1/aws-onion`
- GitHub documentation: `docs/`
- Obsidian learning layer: `00_HOME/` and `01_NOTES/`
- Future book: `book/`
