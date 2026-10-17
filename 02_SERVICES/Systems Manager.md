---
type: service-dna
service: "Systems Manager"
verb: "MANAGE"
object: "Fleet operational actions"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Systems Manager — MANAGE

> [!abstract] LEGO block
> **MANAGE → Fleet operational actions**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **MANAGE** |
| **OBJETO** | Fleet operational actions |
| **RESPONSABILIDAD / WHY** | manage fleet operational actions para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Regional operations service |
| **UPSTREAM (entrada/dependencia)** | IAM + SSM agent + connectivity |
| **DOWNSTREAM (salida/interfaz)** | Managed instances and operational results |
| **RELACIONES / COOPERATES** | EC2, IAM Role, SSM Agent, CloudWatch |
| **NO ES SU TRABAJO** | SSM agent alone doesn't authorize management |
| **EXAM TRIGGER** | Secure management without direct SSH |

## Encaje LEGO
- **Puerto de entrada:** IAM + SSM agent + connectivity.
- **Acción central:** MANAGE sobre Fleet operational actions.
- **Puerto de salida:** Managed instances and operational results.
- **Conexiones laterales:** EC2, IAM Role, SSM Agent, CloudWatch.
- **Regla de descarte:** SSM agent alone doesn't authorize management.

## Pregunta de arquitectura
**¿Qué requisito exige «MANAGE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[Systems Manager - Mental Model]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
