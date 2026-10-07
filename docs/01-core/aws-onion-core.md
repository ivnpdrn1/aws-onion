# Core AWS-Onion Mental Model

## Relationship before memorization

AWS services become easier to remember when each component has a position and a relationship to the others.

A useful networking Onion:

```text
ORIGIN
  ↓
GATEWAY
  ↓
PATH
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

For an Internet-facing EC2 workload:

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
SSH / Apache / Application
```

This is a conceptual dependency/troubleshooting model rather than a literal sequence of independent physical devices.

## Layer associations

**Internet** — external origin/destination.

**Internet Gateway (IGW)** — Internet connectivity doorway for the VPC when addressing and routing support it.

**Route Table** — decides the path for destination networks.

**Subnet** — places resources within a portion of the VPC CIDR.

**NACL** — stateless subnet-level filtering.

**Security Group** — stateful resource/ENI-level filtering.

**ENI** — virtual network interface connecting and identifying the resource in the VPC.

**EC2** — virtual computer running an operating system.

**Application** — software providing the final service, such as SSH, Apache or an application server.

## Public subnet versus public IP

A public subnet is defined by routing that provides a path to an Internet Gateway. An EC2 instance inside it does not necessarily have a public IPv4 address.

Mental rule:

> **Route ≠ Address ≠ Permission**

- Route: is there a path?
- Address: can the destination/source be addressed appropriately?
- Permission: do NACL/SG and other controls allow the traffic?

## Public and private IP

The ENI has private addressing inside the VPC. For direct public IPv4 connectivity, AWS provides public addressing/mapping outside the guest OS.

```text
Internet User
     ↓
Public IPv4
     ↓
AWS / IGW + VPC networking
     ↓
Private IPv4
     ↓
ENI
     ↓
EC2
```

The VPC should not be imagined as a public-IP range. Its CIDR organizes private network address space.

## Routing uses ranges

AWS does not normally require one route per EC2.

```text
10.0.0.0/16 → local
```

can cover many resources.

Mental rule:

> **Route Table finds the network/path; the private IP identifies the destination ENI.**

## CIDR capacity

For a /24:

```text
32 - 24 = 8 host bits
2^8 = 256 total addresses
AWS reserves 5
251 usable
```

AWS reserves five addresses in every subnet. Other AWS resources and ENIs also consume private addresses, so “usable IPs” should not automatically be equated with “number of EC2 instances.”

## Troubleshooting from outside inward

```text
Is connectivity possible?
        ↓
Is there a route?
        ↓
Do subnet controls allow it?
        ↓
Does the Security Group allow it?
        ↓
Are we reaching the correct ENI/IP?
        ↓
Is EC2 healthy?
        ↓
Is the service listening/running?
```

## Core exam phrase

> **ORIGIN → GATEWAY → PATH → SUBNET → RESOURCE → INTERFACE → MACHINE → SERVICE**


## Structural Onion and Functional Onion

AWS-Onion uses two complementary reasoning modes:

| Model | Question |
|---|---|
| **Structural Onion** | What layers exist and how are they related? |
| **Functional Onion** | When an action occurs, what changes, what persists, what is released, and what happens next? |

The lifecycle of EC2 is the first complete Functional Onion example.

> **Structural Onion = WHAT is connected. Functional Onion = WHAT HAPPENS NEXT.**

See [AWS-Onion — Functional Onion Model](FUNCTIONAL-ONION.md).
