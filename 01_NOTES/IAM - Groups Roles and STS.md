---
type: concept
domain: iam
tags: [aws, iam, role, group, sts]
---

# IAM — Groups, Roles and STS

## The distinction

Both Group membership and Role assumption require authorization. The deeper distinction is **identity and credentials**.

### Group

```text
Ivan
 ↓
Developers Group contributes permissions
 ↓
Ivan continues calling AWS as Ivan
```

> **GROUP = same identity + permissions through membership.**

A Group is not an IAM principal and cannot itself assume a Role.

### Role

```text
Principal
 ↓
AssumeRole
 ↓
STS
 ↓
Temporary credentials
 ↓
Role Session
 ↓
AWS
```

> **ROLE = temporary Role session + temporary credentials under the Role identity.**

## Anatomy of a Role

```text
ROLE
├── Trust Policy       → WHO may assume it?
└── Permission Policy  → WHAT may it do?
```

## Group permissions while using a Role

The User's Group permissions are not automatically added to calls made with Role-session credentials.

```text
User credentials → User/Group effective permissions

Role credentials → Role-session effective permissions
```

The Group permissions still exist; they apply when the User acts with the User's own credentials.

## EC2 Role

```text
EC2
 ↓
IAM Role
 ↓
temporary credentials
 ↓
AWS APIs
```

This avoids storing long-lived access keys on the instance.

## Related
- [[Systems Manager - Mental Model]]
- [[AWS-Onion - Mental Model]]
