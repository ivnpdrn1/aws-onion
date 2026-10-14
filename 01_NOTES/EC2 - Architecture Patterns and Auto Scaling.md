---
type: study-guide
domain: compute-architecture
updated: 2026-10-14
tags: [aws, aws-onion, ec2, alb, auto-scaling, cloudwatch, architecture, exam]
---
# EC2 — Architecture Patterns, Scaling and CloudWatch

> **Think like an architect:** requirement → constraint → choice → trade-off. **Traffic does NOT flow through Auto Scaling.**

## The three classroom requirements

| Requirement | First solution | Why not alternatives? |
|---|---|---|
| Short Bash script to initialize EC2 Linux after launch | **EC2 User Data** (often executed once at first boot by cloud-init) | SSH manual is laborious; Systems Manager is useful for fleet/run-command operations; custom AMI is for a reusable prebuilt state |
| Tightly coupled HPC, low latency between nodes | **Cluster Placement Group + compatible EFA** within one AZ | Cluster colocates, EFA supports HPC communication; Spread separates hosts and targets resilience rather than shortest latency |
| Stable base load + short weekly traffic spikes, lowest viable cost | **Commitment for base (RI/Savings Plans) + Spot for flexible bursts** | All On-Demand misses discounts on stable usage; all committed capacity wastes spend during quiet hours; all Spot risks interruption |

## Functional Onion: web traffic vs capacity decisions

```text
DATA PLANE (requests)
Users → ALB listener/rules → Target Group → healthy EC2 instances

CONTROL PLANE (capacity)
Metrics/CloudWatch or schedule/manual action
     → scaling policy / scheduled action
     → Auto Scaling Group desired capacity
     → launch or terminate EC2
     → Target Group registration + health checks
     → ALB routes to healthy, ready targets
```

**ALB:** decides **which** healthy target gets a request. **ASG:** manages **how many** instances should exist. Neither universally requires the other: ALB can front static EC2, and ASG can scale queue workers without an ALB.

### Scaling example (LoB application)
- Weekly baseline: 4 EC2, weekend spike: 12 EC2 for 3 hours.
- Set ASG **min=4**, **desired=4**, **max=12** as illustrative initial settings.
- Use commitment discounts to cover a forecast of at least 4 equivalent EC2 continuously. **Reserved Instances/Savings Plans are billing commitments, not the mechanism that creates EC2.**
- For 8 temporary EC2, consider **Spot** when interruptions can be handled, with On-Demand fallback/availability strategy for critical traffic.
- Auto Scaling Mixed Instances Policy can combine purchase options; capacity-optimized Spot allocation, diversification, multiple instance types/AZ and graceful draining improve robustness.
- If traffic spikes follow a predictable weekly schedule, **Scheduled Scaling** can pre-warm capacity; dynamic policies can handle deviations.
- **ALB + target group** distributes traffic to healthy instances. ASG adds/removes instances; scaling actions take time to launch and pass health checks.

## Why choose the combination? Alternatives

| Approach | Benefit | Trade-off |
|---|---|---|
| All On-Demand | No long-term usage commitment; not Spot-interruptible | Less discount for stable workload |
| All Reserved | Discounted committed usage | Commitment cost remains when usage drops; not a scaling mechanism |
| All Spot | Low price | Can be reclaimed, generally with ~2-minute interruption notice |
| Base commitment + Spot bursts | Optimize predictable base and opportunistic peak | Need interruption tolerance and fallback |
| Base commitment + On-Demand bursts | Stable discounted base, reliable flexible peak | More expensive than Spot when available |
| Savings Plans + elastic capacity | Financial commitment can be flexible across eligible compute | Does not itself reserve capacity |

**EXAM:** The classroom's Reserved + Spot answer presumes the peak is interruption-tolerant. If business-critical sessions cannot tolerate interruptions, prefer On-Demand for the peak or a tested mixed/fallback design.

## CloudWatch: essential or optional for EC2 Auto Scaling?

| Scaling mode | Is CloudWatch metric-based detection necessary? | Mental model |
|---|---|---|
| Manual | **No** | Operator changes desired capacity |
| Scheduled | **No** | Clock triggers a capacity change |
| Dynamic / Target Tracking | **Yes, metrics**; service manages CloudWatch alarms for target tracking | Metrics signal demand; policy adjusts desired capacity |
| Dynamic / Step Scaling | **Typically CloudWatch alarms** | Alarm crossings trigger steps |
| Predictive | **Historical metrics used** | Forecast load and scale ahead |
| ASG replacement of unhealthy EC2 | **No CPU alarm required** | ASG maintains desired healthy capacity |

CloudWatch supplies metrics/alarms; **ASG executes** launching/termination; **ALB routes** requests. Dynamic target tracking does not require hand-created CloudWatch alarms.

## Capacity controls

| Setting | Meaning |
|---|---|
| **Min** | Floor for group size |
| **Desired** | Current capacity ASG tries to maintain |
| **Max** | Ceiling of scaling |
| **Scaling Policy** | Logic to change desired capacity |
| **Target Group health** | When ALB may send requests to an instance |

## Economic vs placement vs operations (Horizontal Onion)

| Dimension | Examples | Do not confuse |
|---|---|---|
| Buying / billing | On-Demand, RI, Savings Plans, Spot | A RI/Savings Plan doesn't automatically add an EC2 |
| Hardware isolation | Dedicated Instance, Dedicated Host | Both isolated; Host adds physical host visibility/affinity, socket/core licensing |
| Proximity vs separation | Cluster, Spread, Partition | Cluster = HPC locality; Spread = isolated individual hosts; Partition = separate fault groups |
| Launch preparation | AMI, User Data | AMI supplies starting image; User Data supplies launch-time instructions |
| Instance discovery | Instance Metadata (IMDSv2) | Metadata reports instance facts; isn't User Data |
| Network identity | ENI + private IP; optional public IP/EIP | A route does not assign an IP or authorize traffic |
| Private egress | Public NAT Gateway in public subnet; private RT→NAT, public RT→IGW | NAT is a resource, not a setting on EC2 ENI |

## Dedicated Instance vs Dedicated Host — exam nuance

Dedicated Instance = single-tenant physical hardware for your account without host-level visibility/selection. Dedicated Host = allocated whole physical host with host ID, socket/core visibility and control/affinity suitable for certain BYOL restrictions. AWS still manages hardware in both cases. Evaluate vendor licensing terms; not every license is transferable.

## Memory anchors (without memorizing isolated service names)

1. **WHO STARTS?** Users request service or EC2 starts work.
2. **WHO ROUTES REQUESTS?** ALB picks healthy targets.
3. **WHO MEASURES?** CloudWatch tracks metrics.
4. **WHO DECIDES CAPACITY?** Scaling policy / schedule / operator.
5. **WHO CHANGES INSTANCE COUNT?** ASG.
6. **WHO PAYS AND HOW?** RI / Savings Plans / On-Demand / Spot selection.

```text
ALB = which EC2?
ASG = how many EC2?
CloudWatch = what is happening?
Scaling Policy = under what rule do we resize?
RI / Savings Plans = how do we discount stable spend?
Spot = what interruptible spare capacity can we exploit?
```

## Quiz (with answers)
1. Bash on first EC2 launch? → User Data.
2. Closely coupled HPC? → Cluster Placement Group + EFA (supported instance types).
3. Is ALB required for Auto Scaling? → No.
4. Is CloudWatch required for manually changing desired capacity? → No.
5. Dynamic target tracking: create alarm yourself? → Normally no; Auto Scaling manages it.
6. Predictable weekly burst? → Scheduled Scaling can start capacity before peak; dynamic scaling can supplement.
7. Spot interrupted during customer peak? → Need tolerated interruption / fallback On-Demand.
8. Physical core licensing? → Dedicated Host.
9. Who receives HTTP requests: ASG or ALB? → ALB → EC2 targets, not through ASG.

## Target Group — key missing link

See [[ALB - Target Groups and Auto Scaling Functional Onion]] for the full table: client → ALB listener/rules → logical Target Group selection → EC2 ENI/app, and independent CloudWatch → policy → ASG → registration → health checks. **Target Group is not a physical network hop.**

## Cross-links
- [[EC2 - Pricing Options]]
- [[EC2 - Networking and Connectivity]]
- [[EC2 - Instance Lifecycle]]
- [[CloudWatch - Mental Model]]
- [[NAT Gateway - Private to Public Functional Onion]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[AWS-Onion - Functional Onion]]

## Official AWS reading
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-simple-step.html
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-scaling-target-tracking.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-purchase-options.html
