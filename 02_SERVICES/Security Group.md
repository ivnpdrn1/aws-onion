---
type: service-dna
service: "Security Group"
verb: "AUTHORIZE"
object: "Inbound/outbound network flows (stateful)"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Security Group — AUTHORIZE

> [!abstract] LEGO block
> **AUTHORIZE → Inbound/outbound network flows (stateful)**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **AUTHORIZE** |
| **OBJETO** | Inbound/outbound network flows (stateful) |
| **RESPONSABILIDAD / WHY** | authorize inbound/outbound network flows (stateful) para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Resource/ENI association |
| **UPSTREAM (entrada/dependencia)** | Route and attempted communication |
| **DOWNSTREAM (salida/interfaz)** | Permitted traffic to or from resource interface |
| **RELACIONES / COOPERATES** | ENI, EC2, ALB |
| **NO ES SU TRABAJO** | Not a route table; cannot explicitly deny rules |
| **EXAM TRIGGER** | Stateful resource filtering |

## Encaje LEGO
- **Puerto de entrada:** Route and attempted communication.
- **Acción central:** AUTHORIZE sobre Inbound/outbound network flows (stateful).
- **Puerto de salida:** Permitted traffic to or from resource interface.
- **Conexiones laterales:** ENI, EC2, ALB.
- **Regla de descarte:** Not a route table; cannot explicitly deny rules.

## Pregunta de arquitectura
**¿Qué requisito exige «AUTHORIZE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
