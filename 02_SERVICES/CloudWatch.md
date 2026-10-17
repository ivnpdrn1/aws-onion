---
type: service-dna
service: "CloudWatch"
verb: "OBSERVE"
object: "Metrics, logs, alarms"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# CloudWatch — OBSERVE

> [!abstract] LEGO block
> **OBSERVE → Metrics, logs, alarms**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **OBSERVE** |
| **OBJETO** | Metrics, logs, alarms |
| **RESPONSABILIDAD / WHY** | observe metrics, logs, alarms para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Regional telemetry for many AWS resources |
| **UPSTREAM (entrada/dependencia)** | Metric/log producers such as EC2 and ALB |
| **DOWNSTREAM (salida/interfaz)** | Alarms, dashboards, scaling policies, notifications |
| **RELACIONES / COOPERATES** | EC2, ALB, ASG, SNS |
| **NO ES SU TRABAJO** | Does not itself launch EC2 or route requests |
| **EXAM TRIGGER** | Metrics, monitoring, alarms, troubleshooting |

## Encaje LEGO
- **Puerto de entrada:** Metric/log producers such as EC2 and ALB.
- **Acción central:** OBSERVE sobre Metrics, logs, alarms.
- **Puerto de salida:** Alarms, dashboards, scaling policies, notifications.
- **Conexiones laterales:** EC2, ALB, ASG, SNS.
- **Regla de descarte:** Does not itself launch EC2 or route requests.

## Pregunta de arquitectura
**¿Qué requisito exige «OBSERVE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[CloudWatch - Mental Model]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
