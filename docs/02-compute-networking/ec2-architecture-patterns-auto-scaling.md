# EC2 Architecture Patterns — decision matrix & functional onion

This document supplements the existing EC2 [pricing](./ec2-pricing-options.md), [networking](./ec2-networking.md), [lifecycle](./ec2-instance-lifecycle.md), and [NAT functional onion](./nat-gateway-functional-onion.md) notes.

## Decision table
| Requirement | Solution | Why |
|---|---|---|
| Small Bash initialization at EC2 launch | User Data | Simple first-boot automation |
| Closely coupled HPC with very low latency | Single-AZ Cluster Placement Group + supported EFA | Physical proximity + HPC network capabilities |
| Continuous base + weekly spikes | Savings Plans or RI discounts for base; Spot for interruption-tolerant peaks; On-Demand fallback | Different purchasing strategies for different workloads |

## Separate data and control planes
```text
REQUEST PATH:   User → ALB → Target Group → Healthy EC2
CAPACITY PATH:  CloudWatch metric / scheduled time / manual command
                    → policy or scaling action → ASG desired capacity
                    → EC2 created or removed → target health → ALB distributes
```

The request path **does not pass through** an Auto Scaling Group. ALB is not required for ASG (for example, queue workers); ASG is not required for ALB (fixed target EC2).

## CloudWatch dependency by mode
| Mode | CloudWatch necessary for its scaling trigger? |
|---|---|
| Manual | No |
| Scheduled | No |
| Target tracking dynamic | CloudWatch metrics and AWS-managed alarms |
| Step scaling dynamic | Typically CloudWatch alarms |
| Predictive | Uses historical monitored metrics |
| Replace unhealthy EC2 to restore desired capacity | No CPU threshold alarm |

Auto Scaling Group maintains **min, desired, max**; scaling policy decides changes; ALB chooses healthy backends. Target tracking automatically manages CloudWatch alarms.

## Example: base 4, Friday peak 12
Commitment discount may cover 4 continuously used instances. ASG can increase desired capacity up to 12 during the peak. Spot lowers incremental cost but may be interrupted; an interruption-aware application or On-Demand fallback is required for customer-critical workloads. Scheduled scaling can prepare for known weekly peaks; dynamic scaling reacts to actual metrics.

## Economic, isolation, networking and launch concepts
- **Dedicated Instance:** physical isolation of EC2 tenancy; AWS chooses host.
- **Dedicated Host:** whole identifiable physical server with core/socket visibility and placement affinity for eligible licenses.
- **AMI:** launch image/base. **User Data:** instructions for first boot. **Metadata (IMDSv2):** EC2 queries information about itself.
- **Private IP** identifies VPC interface. **Auto-assigned Public IP** may change after stop/start. **Elastic IP** persists in account until released; public IPv4 addresses incur charges.
- **Cluster** = colocate for latency. **Partition** = separate hardware groups. **Spread** = distinct hardware for individual instances.
- **NAT Gateway public (zonal)** belongs to public subnet, but private subnet's route points to it; public subnet route leads to IGW. NAT is a resource, not an ENI setting.

## Exam mnemonic
**Which? ALB. How many? ASG. What is happening? CloudWatch. When to change? Scaling policy/schedule. What pricing risk? RI/Savings Plans/Spot/On-Demand.**

Canonical Obsidian note: [EC2 — Architecture Patterns and Auto Scaling](../../01_NOTES/EC2%20-%20Architecture%20Patterns%20and%20Auto%20Scaling.md).

Official docs: https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html
