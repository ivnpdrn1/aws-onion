# AWS Systems Manager

## Core idea

AWS Systems Manager (SSM) provides centralized operational management for EC2 and other managed nodes.

```text
Administrator
      ↓
Systems Manager
      ↓
SSM Agent
      ↓
EC2 / Operating System
```

## Three essential pieces

**SSM Agent** — software inside the managed machine that communicates with Systems Manager.

**IAM Role** — gives the instance/agent the required AWS permissions.

**Connectivity** — provides a network path to the required Systems Manager service endpoints.

Mental phrase:

```text
Agent = communicates
Role  = authorizes
Network = provides the path
SSM = manages
```

## Managed Node

When Systems Manager can successfully manage a machine, it appears as a managed node. Useful indicators include platform, OS, running state, agent version and online/ping status.

## Session Manager

Traditional SSH:

```text
Admin
  ↓
network path
  ↓
SG inbound TCP 22
  ↓
ENI
  ↓
EC2
  ↓
SSH
```

Session Manager:

```text
Admin
  ↓
AWS Systems Manager
  ↓
SSM control path
  ↓
SSM Agent
  ↓
EC2
```

The instance still needs connectivity to the appropriate Systems Manager endpoints. Session Manager does not eliminate networking; it eliminates the need for a direct inbound SSH management path.

## Private connectivity

Systems Manager can be reached through suitable network connectivity, including VPC endpoints for relevant SSM services where appropriate.

## Important capabilities

- Session Manager — interactive administration.
- Fleet Manager — centralized managed-node view.
- Run Command — remote command execution.
- Patch Manager — patch operations.
- Inventory — machine/software information.
- Automation — operational runbooks.
- State Manager — desired configuration/state.
- Parameter Store — configuration values/parameters.

## Systems Manager versus EC2 Instance Connect Endpoint

```text
EIC Endpoint
→ “I specifically want SSH to a private EC2.”

Session Manager
→ “I want administrative access without exposing inbound SSH.”
```

## Systems Manager versus workflow orchestrators

Systems Manager primarily orchestrates **systems operations**.

```text
n8n             → application/API/business workflows
Step Functions  → AWS/application workflows
Systems Manager → machine/infrastructure operations
```

## Exam association

When the requirement says:

- manage EC2 at scale
- patch machines
- execute remote commands
- inventory machines/software
- administer without opening inbound SSH
- centralized operational management

associate it with **AWS Systems Manager**.
