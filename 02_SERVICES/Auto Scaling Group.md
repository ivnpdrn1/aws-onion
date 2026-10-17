---
type: service-dna
service: "Auto Scaling Group"
verb: "ADJUST"
object: "EC2 fleet desired/min/max capacity"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Auto Scaling Group — ADJUST

> [!abstract] LEGO block
> **ADJUST → EC2 fleet desired/min/max capacity**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **ADJUST** |
| **OBJETO** | EC2 fleet desired/min/max capacity |
| **RESPONSABILIDAD / WHY** | adjust ec2 fleet desired/min/max capacity para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Regional / multiple AZs |
| **UPSTREAM (entrada/dependencia)** | Scaling policy, schedule, health replacements |
| **DOWNSTREAM (salida/interfaz)** | EC2 launch/terminate; register targets to TG |
| **RELACIONES / COOPERATES** | CloudWatch, launch template, EC2, target group |
| **NO ES SU TRABAJO** | Not a traffic router or data-plane hop |
| **EXAM TRIGGER** | Scale out/in and maintain healthy capacity |

## Encaje LEGO
- **Puerto de entrada:** Scaling policy, schedule, health replacements.
- **Acción central:** ADJUST sobre EC2 fleet desired/min/max capacity.
- **Puerto de salida:** EC2 launch/terminate; register targets to TG.
- **Conexiones laterales:** CloudWatch, launch template, EC2, target group.
- **Regla de descarte:** Not a traffic router or data-plane hop.

## Pregunta de arquitectura
**¿Qué requisito exige «ADJUST» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Architecture Patterns and Auto Scaling]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
