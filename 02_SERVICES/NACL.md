---
type: service-dna
service: "NACL"
verb: "FILTER"
object: "Subnet traffic (stateless rules)"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# NACL — FILTER

> [!abstract] LEGO block
> **FILTER → Subnet traffic (stateless rules)**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **FILTER** |
| **OBJETO** | Subnet traffic (stateless rules) |
| **RESPONSABILIDAD / WHY** | filter subnet traffic (stateless rules) para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Subnet boundary |
| **UPSTREAM (entrada/dependencia)** | Network traffic at subnet boundary |
| **DOWNSTREAM (salida/interfaz)** | Allowed/rejected packets with return-traffic rules |
| **RELACIONES / COOPERATES** | Subnet, security group |
| **NO ES SU TRABAJO** | Not stateful; return traffic requires explicit rules |
| **EXAM TRIGGER** | Stateless subnet filtering |

## Encaje LEGO
- **Puerto de entrada:** Network traffic at subnet boundary.
- **Acción central:** FILTER sobre Subnet traffic (stateless rules).
- **Puerto de salida:** Allowed/rejected packets with return-traffic rules.
- **Conexiones laterales:** Subnet, security group.
- **Regla de descarte:** Not stateful; return traffic requires explicit rules.

## Pregunta de arquitectura
**¿Qué requisito exige «FILTER» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
