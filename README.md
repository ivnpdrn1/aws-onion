# AWS-Onion

**AWS-Onion** is a relationship-first framework for learning AWS architecture, preparing for AWS certifications, and eventually developing a book about how to build a durable AWS mental model.

> AWS is easier to understand when services are learned as connected layers rather than isolated definitions.

## The Onion idea

A recurring networking example:

```text
Internet
   ↓
Internet Gateway
   ↓
Route Table / Subnet
   ↓
NACL
   ↓
Security Group
   ↓
ENI / Private IP
   ↓
EC2
   ↓
Application / Service
```

This is a **conceptual dependency and troubleshooting model**, not a literal claim that every packet traverses independent physical boxes in exactly this order.

Each layer should answer:

1. What problem does it solve?
2. What is outside this layer?
3. What is inside it?
4. What does it depend on?
5. What does it provide to the next layer?
6. Which identity/principal needs permission?
7. What network path is required?
8. What association matters for the certification exam?

## Certification tracks

AWS-Onion is being built around:

- **AWS Solutions Architect** — architecture, networking, compute, storage, resilience, security and design.
- **AWS Developer** — application services, APIs, events, serverless, SDK/CLI, IAM and deployment.
- **AWS AI** — AI services, Bedrock, agents, data, security, responsible AI and AI workload architecture.

See [Certification Roadmap](docs/00-roadmap/certification-roadmap.md).

## LEGO normalization — service-first composition

- [AWS-Onion LEGO Architecture Map](00_HOME/AWS-Onion%20-%20LEGO%20Architecture%20Map.md)
- [Service DNA catalog: 18 AWS building blocks](02_SERVICES/README.md)
- [Reusable architecture blueprints](03_BLUEPRINTS/README.md)

**Standard:** VERB → OBJECT → RESPONSIBILITY → RELATIONSHIPS → ARCHITECTURE DECISION. Each service has upstream/downstream interfaces and scope; designs combine pieces by requirement and keep DNS, data, control and observability planes distinct. Existing notes remain under `01_NOTES` and detailed manuals under `docs/`.

- [Scaling Up vs Scaling Out — Stateful / Stateless](01_NOTES/Scaling%20Up%20vs%20Scaling%20Out%20-%20Stateful%20Stateless%20Functional%20Onion.md)

## Current knowledge base

- [Verbo rector y Service DNA](01_NOTES/AWS-Onion%20-%20Verbo%20Rector%20y%20Service%20DNA.md)
- [Core AWS-Onion Mental Model](docs/01-core/aws-onion-core.md)
- [Vertical, Horizontal & Functional Axes](docs/01-core/VERTICAL-HORIZONTAL-ONION.md)
- [Functional Onion Model](docs/01-core/FUNCTIONAL-ONION.md)
- [Route 53 — DNS vs Load Balancing (Obsidian note)](01_NOTES/Route%2053%20-%20DNS%20vs%20Load%20Balancing%20Multi-Region%20Onion.md)
- [EC2 Networking & Connectivity](docs/02-compute-networking/ec2-networking.md)
- [EC2 Instance Lifecycle](docs/02-compute-networking/ec2-instance-lifecycle.md)
- [AWS Nitro System & Nitro Enclaves](docs/02-compute-networking/nitro-system-enclaves.md)
- [EC2 Pricing Options](docs/02-compute-networking/ec2-pricing-options.md)
- [IAM Groups, Roles & STS](docs/03-iam/roles-groups-sts.md)
- [Systems Manager](docs/04-operations/systems-manager.md)
- [CloudWatch](docs/04-operations/cloudwatch.md)
- [Certification Roadmap](docs/00-roadmap/certification-roadmap.md)
- [Future Book](book/BOOK-VISION.md)

## Memory phrases

```text
Structural Onion = WHAT is connected.
Vertical Onion = BETWEEN layers.
Horizontal Onion = WITHIN a layer.
Functional Onion = THROUGH time / WHAT HAPPENS NEXT.

Exam reconstruction = WHERE → WHAT INSIDE → WHAT NEXT.

Actions / configuration descend.
Status / metrics / errors / evidence return upward.

Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.
Resource lifecycle ≠ Pricing commitment lifecycle.

Route 53 = WHERE endpoint/region. ELB = WHICH target in-region. ASG = HOW MANY. EC2 = EXECUTE.
DNS answers queries; ELB is inline with application traffic.

Route ≠ Address ≠ Permission
Price ≠ Capacity ≠ Tenancy ≠ Billing

NACL protects the subnet.
Security Group protects access to the resource/interface.
ENI connects and identifies the machine.
EC2 runs the operating system.
The application provides the service.

On-Demand = flexibility.
Savings Plan = commit $/hour.
Reserved Instance = configuration commitment.
Spot = spare + interruptible.
Capacity Reservation = capacity assurance.
Dedicated Instance = isolation.
Dedicated Host = host-level control.

GROUP = same user identity + permissions through membership.
ROLE  = temporary role session + temporary credentials.

SSM Agent communicates.
IAM Role authorizes.
Network provides the path.
Systems Manager manages.
```

## Long-term objective

The repository will evolve from certification notes into a structured teaching framework and, eventually, a book.

Working concept:

**AWS-Onion: Understanding AWS Through Relationships Between Layers**

The objective is not merely to remember what each AWS service does. It is to be able to **reconstruct an architecture from the relationships among its layers**.

## EC2 Auto Scaling — relación integrada
- [EC2 Auto Scaling — MAINTAIN + ADJUST](01_NOTES/EC2%20Auto%20Scaling%20-%20Maintain%20Adjust%20Functional%20Onion.md)
**Launch Template DEFINE → ASG MAINTAIN → CloudWatch OBSERVE / Scaling Policy DEFINE RESPONSE → ASG ADJUST → EC2 EXECUTE.** ELB DISTRIBUTE; Target Group GROUP/CHECK. Los health checks EC2 y ELB tienen alcances distintos. Consultar la nota para las modalidades manual, dynamic, predictive y scheduled y las distinciones Scaling Up/Out y stateful/stateless.

## EC2 User Data — LEGO de arranque y servidor web
- [Análisis línea por línea: Apache, Bash, IMDSv2, HTML y alternativas](01_NOTES/EC2%20User%20Data%20-%20Bootstrap%20Apache%20IMDSv2%20LEGO%20Onion.md)
- [Ficha Service DNA: EC2 User Data BOOTSTRAP](02_SERVICES/EC2%20User%20Data%20-%20Bootstrap%20LEGO%20Service%20DNA.md)

**Cadena LEGO:** Launch Template DEFINE → ASG MAINTAIN → EC2 EXECUTE → User Data BOOTSTRAP → Apache SERVE → Target Group CHECK → ALB DISTRIBUTE; IMDSv2 DISCOVER contextualiza la instancia y CloudWatch OBSERVE retroalimenta escalado.

## Resiliencia — AWS-ONION-LEGO (2026-10-23)
- [High Availability vs Fault Tolerance vs Durability — explicación integrada](01_NOTES/High%20Availability%20Fault%20Tolerance%20Durability%20-%20AWS%20Onion%20LEGO.md)
- [Reliability LEGO — ficha de ensamblaje](02_SERVICES/Reliability%20-%20High%20Availability%20Fault%20Tolerance%20Durability%20LEGO.md)
- [Semilla de capítulo del libro — Cuando falla una pieza](book/CHAPTER-SEED-Resilience-When-One-LEGO-Fails.md)

**Memory phrase:** EC2 EXECUTE; ELB DISTRIBUTE; ASG MAINTAIN; Multi-AZ ISOLATE failure; HA MINIMIZE downtime; FT RESIST specified faults; durability PRESERVE data.
