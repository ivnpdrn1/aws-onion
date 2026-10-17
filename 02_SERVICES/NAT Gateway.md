---
type: service-dna
service: "NAT Gateway"
verb: "TRANSLATE"
object: "Outbound connections from private IPv4 resources"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# NAT Gateway — TRANSLATE

> [!abstract] LEGO block
> **TRANSLATE → Outbound connections from private IPv4 resources**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **TRANSLATE** |
| **OBJETO** | Outbound connections from private IPv4 resources |
| **RESPONSABILIDAD / WHY** | translate outbound connections from private ipv4 resources para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | AZ-specific NAT Gateway deployment |
| **UPSTREAM (entrada/dependencia)** | Private subnet route to NAT |
| **DOWNSTREAM (salida/interfaz)** | Public NAT GW → IGW → internet |
| **RELACIONES / COOPERATES** | Subnet, route table, IGW |
| **NO ES SU TRABAJO** | Not a general-purpose inbound internet entry for private EC2 |
| **EXAM TRIGGER** | Private subnet outbound access |

## Encaje LEGO
- **Puerto de entrada:** Private subnet route to NAT.
- **Acción central:** TRANSLATE sobre Outbound connections from private IPv4 resources.
- **Puerto de salida:** Public NAT GW → IGW → internet.
- **Conexiones laterales:** Subnet, route table, IGW.
- **Regla de descarte:** Not a general-purpose inbound internet entry for private EC2.

## Pregunta de arquitectura
**¿Qué requisito exige «TRANSLATE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[NAT Gateway - Private to Public Functional Onion]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
