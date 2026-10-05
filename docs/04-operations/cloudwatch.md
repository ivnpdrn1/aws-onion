# Amazon CloudWatch

## Core idea

CloudWatch is the observability/instrument-panel layer for AWS resources and applications.

Mental association:

> **CloudWatch = AWS dashboard/instrument panel.**

AWS services publish or expose metrics in service namespaces such as EC2, EBS, Lambda, load balancing, API Gateway and others.

## Basic relationship

```text
AWS Resource / Service
        ↓
      Metric
        ↓
    CloudWatch
        ↓
 Graph / Alarm
        ↓
Action / Notification
```

## EC2 examples

A common EC2 metric is:

```text
CPUUtilization
```

Status-related metrics include concepts corresponding to system and instance status-check failures.

This connects CloudWatch to the EC2 mental model:

```text
AWS underlying infrastructure
        ↓
System Status Check

Guest EC2 / OS
        ↓
Instance Status Check

Both can feed operational visibility
        ↓
CloudWatch
```

## Exam association

When the requirement is about metrics, alarms, operational visibility or monitoring AWS resources, CloudWatch should be one of the first services considered.

The Onion question is:

> **What layer produces the signal, and what should happen when CloudWatch observes it?**
