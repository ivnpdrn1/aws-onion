---
type: concept
domain: observability
tags: [aws, cloudwatch, monitoring]
---

# CloudWatch — Mental Model

> **CloudWatch = AWS dashboard / instrument panel.**

```text
AWS Service
    ↓
Metric
    ↓
CloudWatch
    ↓
Graph / Alarm
    ↓
Action / Notification
```

For EC2, metrics such as CPU utilization and status-check signals provide operational visibility.

## Onion relationship

CloudWatch is not simply “another network layer.” It is an **observability layer looking across other layers**.

The important question becomes:

> Which layer produced the signal, and what should happen when CloudWatch observes it?

## Related
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - Mental Model]]
