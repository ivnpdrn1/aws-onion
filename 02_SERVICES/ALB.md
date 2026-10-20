---
type: service-dna
service: "ALB"
verb: "DISTRIBUTE"
object: "HTTP/HTTPS requests"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# ALB — DISTRIBUTE

> [!abstract] LEGO block
> **DISTRIBUTE → HTTP/HTTPS requests**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **DISTRIBUTE** |
| **OBJETO** | HTTP/HTTPS requests |
| **RESPONSABILIDAD / WHY** | distribute http/https requests para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Regional / multiple AZs |
| **UPSTREAM (entrada/dependencia)** | Client following DNS answer |
| **DOWNSTREAM (salida/interfaz)** | Listener rules → Target Group → healthy registered target |
| **RELACIONES / COOPERATES** | Route 53, EC2, ASG, CloudWatch, SG |
| **NO ES SU TRABAJO** | Cannot load balance targets across AWS regions in one ALB |
| **EXAM TRIGGER** | Host/path routing, HTTP/HTTPS balancing |

## Encaje LEGO
- **Puerto de entrada:** Client following DNS answer.
- **Acción central:** DISTRIBUTE sobre HTTP/HTTPS requests.
- **Puerto de salida:** Listener rules → Target Group → healthy registered target.
- **Conexiones laterales:** Route 53, EC2, ASG, CloudWatch, SG.
- **Regla de descarte:** Cannot load balance targets across AWS regions in one ALB.

## Pregunta de arquitectura
**¿Qué requisito exige «DISTRIBUTE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[ALB - Target Groups and Auto Scaling Functional Onion]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]

## Scaling Up vs Scaling Out — relación incorporada
- [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]]
- **ENLARGE**: más recursos para una sola instancia; **REPLICATE**: más instancias para trabajo distribuible.
- **EXTERNALIZE STATE** cuando el estado local impida reemplazar instancias; opciones por requisito: [[Amazon EFS]], [[Amazon RDS]], [[Amazon DynamoDB]].
- **ALB DISTRIBUTE**, **CloudWatch OBSERVE**, **ASG ADJUST**, **EC2 EXECUTE**: funciones complementarias, no saltos seriales del tráfico.

## EC2 Auto Scaling — relación integrada
- [[EC2 Auto Scaling - Maintain Adjust Functional Onion]]
**Launch Template DEFINE → ASG MAINTAIN → CloudWatch OBSERVE / Scaling Policy DEFINE RESPONSE → ASG ADJUST → EC2 EXECUTE.** ELB DISTRIBUTE; Target Group GROUP/CHECK. Los health checks EC2 y ELB tienen alcances distintos. Consultar la nota para las modalidades manual, dynamic, predictive y scheduled y las distinciones Scaling Up/Out y stateful/stateless.
