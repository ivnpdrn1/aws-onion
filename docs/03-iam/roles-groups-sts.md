# IAM Groups, Roles & STS

## Why Group versus Role can be confusing

Both involve authorization. Someone may authorize a User to belong to a Group, and someone may authorize a principal to assume a Role.

Therefore, “someone authorized it” is not the useful differentiator.

The deeper distinction is **identity and credentials**.

## IAM Group

A Group organizes IAM Users and lets policies contribute permissions to those users.

```text
Developers Group
     ↓
   Ivan
     ↓
Ivan still calls AWS as Ivan
```

The User continues using the User's identity/credentials.

Mental rule:

> **GROUP = same identity + permissions through membership.**

A Group is not an IAM principal and cannot itself assume a Role.

## IAM Role

A Role is an assumable IAM identity.

It has two key sides:

```text
IAM ROLE
 ├── Trust Policy
 │      └── WHO may assume it?
 │
 └── Permission Policies
        └── WHAT may it do?
```

A useful shorthand is:

> **Role ≈ assumable identity/principal + permissions**

More precisely:

> **ROLE = Trust Policy + Permission Policies**

## What “AssumeRole” means

Assuming a Role is not merely adding more permanent permissions to the original User.

```text
IAM User / Principal
        ↓
    AssumeRole
        ↓
       STS
        ↓
Temporary Credentials
        ↓
   Role Session
        ↓
     AWS APIs
```

AWS Security Token Service (STS) provides temporary credentials for the Role session, including an Access Key ID, Secret Access Key, Session Token and expiration.

Mental rule:

> **Assume Role = use a temporary credential set to act under the Role session.**

## Group versus Role

```text
GROUP

Ivan
  ↓
Group contributes permissions
  ↓
Ivan uses Ivan's credentials
  ↓
AWS


ROLE

Ivan
  ↓
AssumeRole
  ↓
STS
  ↓
temporary Role credentials
  ↓
Role Session
  ↓
AWS
```

This leads to the strongest mental distinction developed in the study sessions:

> **GROUP = MISMA IDENTIDAD + permisos por pertenencia.**

> **ROLE = OTRAS CREDENCIALES + sesión temporal bajo la identidad del Role.**

## Do Group permissions combine with Role permissions?

Not automatically.

When a User makes calls using the temporary Role credentials, AWS evaluates those calls as the Role session. The User's Group permissions are not automatically added to the Role permissions.

```text
USER CREDENTIALS
      ↓
User/Group effective permissions


ROLE TEMPORARY CREDENTIALS
      ↓
Role-session effective permissions
```

The Group permissions have not been deleted or “turned off.” They remain relevant when the User acts with the User's own credentials.

## EC2 Roles

An EC2 instance can use an IAM Role rather than storing long-lived access keys.

```text
EC2
 ↓
IAM Role
 ↓
temporary credentials
 ↓
AWS APIs
```

For Systems Manager:

```text
EC2
 ├── SSM Agent → communicates
 └── IAM Role  → authorizes
        ↓
Systems Manager
```

The Role is not “Systems Manager entering EC2.” The instance/agent uses the permissions associated with its Role to interact with AWS services.

## Exam memory

```text
GROUP → membership → same identity → permissions through group policies

ROLE  → trust → AssumeRole → STS → temporary credentials → Role Session
```
