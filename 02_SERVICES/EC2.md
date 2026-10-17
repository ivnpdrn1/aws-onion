---
type: service-dna
service: "EC2"
verb: "EXECUTE"
object: "Application compute workload"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# EC2 — EXECUTE

> [!abstract] LEGO block
> **EXECUTE → Application compute workload**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **EXECUTE** |
| **OBJETO** | Application compute workload |
| **RESPONSABILIDAD / WHY** | execute application compute workload para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Instance in one AZ |
| **UPSTREAM (entrada/dependencia)** | Request arriving through load balancer / direct approved path |
| **DOWNSTREAM (salida/interfaz)** | App processes and data stores |
| **RELACIONES / COOPERATES** | ENI, SG, AMI, EBS, ASG, CloudWatch |
| **NO ES SU TRABAJO** | Does not automatically add EC2 capacity alone |
| **EXAM TRIGGER** | Choose instance computing / AMI / AZ |

## Encaje LEGO
- **Puerto de entrada:** Request arriving through load balancer / direct approved path.
- **Acción central:** EXECUTE sobre Application compute workload.
- **Puerto de salida:** App processes and data stores.
- **Conexiones laterales:** ENI, SG, AMI, EBS, ASG, CloudWatch.
- **Regla de descarte:** Does not automatically add EC2 capacity alone.

## Pregunta de arquitectura
**¿Qué requisito exige «EXECUTE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Instance Lifecycle]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
