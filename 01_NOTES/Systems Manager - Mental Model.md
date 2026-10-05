---
type: concept
domain: operations
tags: [aws, ssm, session-manager, operations]
---

# Systems Manager — Mental Model

## Essential relationship

```text
Administrator
      ↓
Systems Manager
      ↓
SSM Agent
      ↓
EC2 / OS
```

Three pieces:

> **Agent communicates. Role authorizes. Network provides the path. Systems Manager manages.**

The instance still needs connectivity to the required Systems Manager endpoints.

## Session Manager

Session Manager avoids the need for a direct inbound SSH management path.

It does **not** eliminate networking.

## Important capabilities

Session Manager, Fleet Manager, Run Command, Patch Manager, Inventory, Automation, State Manager and Parameter Store.

## Exam association

When the requirement mentions centralized EC2 management, remote commands, patching, inventory, or administrative access without inbound SSH, think **Systems Manager**.

## Related
- [[IAM - Groups Roles and STS]]
- [[EC2 - Networking and Connectivity]]
