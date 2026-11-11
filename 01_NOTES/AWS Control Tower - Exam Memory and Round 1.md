---
type: exam-review
project: AWS-Onion
topic: AWS Control Tower
date: 2026-11-11
---
# AWS Control Tower — Exam Memory and Round 1

## Evaluación confirmada
**Ronda 1: 10/10, 10 preguntas respondidas, todos los ítems correctos.** Es dominio en el formato de elección múltiple presentado, no demostración todavía de diseño abierto.

## Cobertura efectiva
1. Elegir Control Tower en vez de solo Organizations para landing zone multicuentas.
2. Landing Zone como baseline, no VPC ni subnet.
3. Log Archive PRESERVE registros.
4. Audit INSPECT / supervisión.
5. SCP como medida preventiva de APIs prohibidas.
6. Organizations, Identity Center y Config con responsabilidades distintas.
7. Account Factory PROVISION de forma estandarizada.
8. Config DETECT configuraciones no conformes.
9. Detective no implica remediation automática.
10. Proactive / CloudFormation Hooks VALIDATE durante provisión.

## Recuperación rápida: pares que no debemos confundir
- Organizations ORGANIZE vs Control Tower ORCHESTRATE.
- SCP LIMIT vs IAM AUTHORIZE vs Identity Center ACCESS / ASSIGN.
- CloudTrail RECORD events vs Config EVALUATE configuration.
- Log Archive PRESERVE vs Audit INSPECT.
- Preventive BLOCK vs Detective DETECT vs Proactive VALIDATE before deployment.
- Account Factory PROVISION accounts vs ASG MAINTAIN EC2 instances.

## Nivel 2 futuro (5 escenarios de explicación libre)
Evaluar cinco ejes de 0 a 4 puntos: VERBO/OBJETO, RESPONSABILIDAD, RELACIONES, DECISIÓN, FALLOS/LÍMITES. 20/20 requerido por escenario para avanzar. Situaciones: crear 25 cuentas, elegir Control Tower vs IaC manual, diseñar logging y auditoría, elegir control preventivo/detective/proactivo y razonar límites de gestión y de IAM. No marcar el Nivel 2 completado.

[[AWS Control Tower - Orchestrate Landing Zone LEGO]]
