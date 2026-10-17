---
type: service-dna
service: "Internet Gateway"
verb: "CONNECT"
object: "Internet connectivity for public IPv4 / IPv6 paths"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Internet Gateway — CONNECT

> [!abstract] LEGO block
> **CONNECT → Internet connectivity for public IPv4 / IPv6 paths**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **CONNECT** |
| **OBJETO** | Internet connectivity for public IPv4 / IPv6 paths |
| **RESPONSABILIDAD / WHY** | connect internet connectivity for public ipv4 / ipv6 paths para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Attached to VPC |
| **UPSTREAM (entrada/dependencia)** | Public subnet route to IGW |
| **DOWNSTREAM (salida/interfaz)** | Internet-facing path with appropriate addressing |
| **RELACIONES / COOPERATES** | VPC, route table, SG, NACL |
| **NO ES SU TRABAJO** | Does not make instances public without addressing/routes/policy |
| **EXAM TRIGGER** | Public ingress/egress |

## Encaje LEGO
- **Puerto de entrada:** Public subnet route to IGW.
- **Acción central:** CONNECT sobre Internet connectivity for public IPv4 / IPv6 paths.
- **Puerto de salida:** Internet-facing path with appropriate addressing.
- **Conexiones laterales:** VPC, route table, SG, NACL.
- **Regla de descarte:** Does not make instances public without addressing/routes/policy.

## Pregunta de arquitectura
**¿Qué requisito exige «CONNECT» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
