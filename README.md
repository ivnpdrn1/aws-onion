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

## Current knowledge base

- [Core AWS-Onion Mental Model](docs/01-core/aws-onion-core.md)
- [EC2 Networking & Connectivity](docs/02-compute-networking/ec2-networking.md)
- [EC2 Instance Lifecycle](docs/02-compute-networking/ec2-instance-lifecycle.md)
- [IAM Groups, Roles & STS](docs/03-iam/roles-groups-sts.md)
- [Systems Manager](docs/04-operations/systems-manager.md)
- [CloudWatch](docs/04-operations/cloudwatch.md)
- [Certification Roadmap](docs/00-roadmap/certification-roadmap.md)
- [Future Book](book/BOOK-VISION.md)

## Memory phrases

```text
Actions / configuration descend.
Status / metrics / errors / evidence return upward.

Compute lifecycle ≠ Storage lifecycle ≠ Network-address lifecycle ≠ Physical-host identity.

Route ≠ Address ≠ Permission

NACL protects the subnet.
Security Group protects access to the resource/interface.
ENI connects and identifies the machine.
EC2 runs the operating system.
The application provides the service.

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
