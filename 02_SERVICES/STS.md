---
type: service-dna
service: "STS"
verb: "ISSUE"
object: "Temporary security credentials"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# STS — ISSUE

> [!abstract] LEGO block
> **ISSUE → Temporary security credentials**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **ISSUE** |
| **OBJETO** | Temporary security credentials |
| **RESPONSABILIDAD / WHY** | issue temporary security credentials para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Global AWS service with regional endpoints |
| **UPSTREAM (entrada/dependencia)** | AssumeRole/federation request |
| **DOWNSTREAM (salida/interfaz)** | Role session credentials |
| **RELACIONES / COOPERATES** | IAM roles, callers |
| **NO ES SU TRABAJO** | Does not replace the role's trust/authorization policies |
| **EXAM TRIGGER** | Temporary credentials |

## Encaje LEGO
- **Puerto de entrada:** AssumeRole/federation request.
- **Acción central:** ISSUE sobre Temporary security credentials.
- **Puerto de salida:** Role session credentials.
- **Conexiones laterales:** IAM roles, callers.
- **Regla de descarte:** Does not replace the role's trust/authorization policies.

## Pregunta de arquitectura
**¿Qué requisito exige «ISSUE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[IAM - Groups Roles and STS]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
