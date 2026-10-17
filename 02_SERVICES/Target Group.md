---
type: service-dna
service: "Target Group"
verb: "GROUP"
object: "Registered targets and health configuration"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Target Group — GROUP

> [!abstract] LEGO block
> **GROUP → Registered targets and health configuration**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **GROUP** |
| **OBJETO** | Registered targets and health configuration |
| **RESPONSABILIDAD / WHY** | group registered targets and health configuration para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Regional / associated to load balancer |
| **UPSTREAM (entrada/dependencia)** | ALB listener forwarding action |
| **DOWNSTREAM (salida/interfaz)** | EC2 targets, IP targets, Lambda where supported |
| **RELACIONES / COOPERATES** | ALB, ASG, target health checks |
| **NO ES SU TRABAJO** | Not a physical packet hop; does not provision EC2 |
| **EXAM TRIGGER** | Targets registered, healthy, eligible |

## Encaje LEGO
- **Puerto de entrada:** ALB listener forwarding action.
- **Acción central:** GROUP sobre Registered targets and health configuration.
- **Puerto de salida:** EC2 targets, IP targets, Lambda where supported.
- **Conexiones laterales:** ALB, ASG, target health checks.
- **Regla de descarte:** Not a physical packet hop; does not provision EC2.

## Pregunta de arquitectura
**¿Qué requisito exige «GROUP» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[ALB - Target Groups and Auto Scaling Functional Onion]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
