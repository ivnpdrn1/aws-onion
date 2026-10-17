---
type: service-dna
service: "IAM Role"
verb: "DELEGATE"
object: "Assumable permission-bearing identity"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# IAM Role — DELEGATE

> [!abstract] LEGO block
> **DELEGATE → Assumable permission-bearing identity**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **DELEGATE** |
| **OBJETO** | Assumable permission-bearing identity |
| **RESPONSABILIDAD / WHY** | delegate assumable permission-bearing identity para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | IAM global (role); temporary sessions |
| **UPSTREAM (entrada/dependencia)** | Principal permitted by trust and identity policies |
| **DOWNSTREAM (salida/interfaz)** | STS-issued role session credentials used for actions |
| **RELACIONES / COOPERATES** | IAM policy, STS, EC2 instance profile |
| **NO ES SU TRABAJO** | Not the same as user group membership |
| **EXAM TRIGGER** | AssumeRole and temporary credentials |

## Encaje LEGO
- **Puerto de entrada:** Principal permitted by trust and identity policies.
- **Acción central:** DELEGATE sobre Assumable permission-bearing identity.
- **Puerto de salida:** STS-issued role session credentials used for actions.
- **Conexiones laterales:** IAM policy, STS, EC2 instance profile.
- **Regla de descarte:** Not the same as user group membership.

## Pregunta de arquitectura
**¿Qué requisito exige «DELEGATE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[IAM - Groups Roles and STS]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
