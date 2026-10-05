# AWS-Onion — Book Vision

## Working title

**AWS-Onion: Understanding AWS Through Relationships Between Layers**

## Thesis

AWS is often difficult to learn because hundreds of services are introduced as separate definitions.

AWS-Onion takes the opposite approach:

> **A service becomes understandable when the learner knows what surrounds it, what it depends on, what it protects or enables, and what it provides to the next layer.**

The objective is to develop a mental model that can be reconstructed rather than a catalog that must be memorized.

## Teaching pattern

Each chapter should repeatedly answer:

```text
What is outside this layer?
        ↓
What enters it?
        ↓
What decision/control happens here?
        ↓
What identity has authority?
        ↓
What leaves this layer?
        ↓
What does the next layer expect?
```

## Example: networking Onion

```text
WORLD
  ↓
VPC CONNECTIVITY
  ↓
SUBNET / ROUTING
  ↓
SUBNET SECURITY
  ↓
RESOURCE SECURITY
  ↓
NETWORK IDENTITY
  ↓
COMPUTE
  ↓
APPLICATION
```

which can become:

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
Application
```

## Beyond networking

The Onion should eventually connect multiple dimensions rather than force every AWS service into one linear diagram.

Examples:

### Identity Onion

```text
Principal
   ↓
Authentication
   ↓
Authorization
   ↓
Policy evaluation
   ↓
Role / temporary session
   ↓
AWS Resource/API
```

### Operations Onion

```text
Administrator
   ↓
AWS control service
   ↓
IAM authorization
   ↓
Network connectivity
   ↓
Agent/API
   ↓
Managed resource
```

### Application Onion

Future chapters will connect API Gateway, Lambda, events, queues, databases and observability.

### AI Onion

Future chapters will connect data, model access, Bedrock, agents, tools, IAM, networking, observability, guardrails and cost.

## Certification and book relationship

The certification notes are not separate from the book. They are the laboratory where the mental model is tested.

If an explanation helps answer an exam question **and** helps reconstruct a real architecture, it belongs in AWS-Onion.

## Guiding principle

> **Do not memorize the onion. Understand why each layer needs the next one.**

That relationship is what turns AWS terminology into architecture.
