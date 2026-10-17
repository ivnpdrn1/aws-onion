---
type: service-dna
service: "Route 53"
verb: "RESOLVE"
object: "DNS name"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Route 53 — RESOLVE

> [!abstract] LEGO block
> **RESOLVE → DNS name**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **RESOLVE** |
| **OBJETO** | DNS name |
| **RESPONSABILIDAD / WHY** | resolve dns name para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Global |
| **UPSTREAM (entrada/dependencia)** | Client DNS query |
| **DOWNSTREAM (salida/interfaz)** | Regional ALB DNS name or other eligible endpoint |
| **RELACIONES / COOPERATES** | ELB, health checks |
| **NO ES SU TRABAJO** | Does not proxy HTTP traffic |
| **EXAM TRIGGER** | Multi-region DNS resolution / failover |

## Encaje LEGO
- **Puerto de entrada:** Client DNS query.
- **Acción central:** RESOLVE sobre DNS name.
- **Puerto de salida:** Regional ALB DNS name or other eligible endpoint.
- **Conexiones laterales:** ELB, health checks.
- **Regla de descarte:** Does not proxy HTTP traffic.

## Pregunta de arquitectura
**¿Qué requisito exige «RESOLVE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[Route 53 - DNS vs Load Balancing Multi-Region Onion]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
