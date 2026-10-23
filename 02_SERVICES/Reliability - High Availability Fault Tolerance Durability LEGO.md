---
type: service-dna-ensemble
updated: 2026-10-23
tags: [aws-onion, lego, reliability, certification, book]
---
# Reliability LEGO — ficha de ensamblaje

**No es un AWS Service aislado:** es el **resultado** de montar varias piezas con contratos compatibles.

| Campo | Contrato |
|---|---|
| REQUISITO | Minimizar downtime, tolerar fallo definido o preservar datos |
| VERBO | RESIST / RECOVER / PRESERVE |
| OBJETO | Servicio extremo a extremo y sus datos |
| RESPONSABILIDAD | Evitar puntos únicos de fallo dentro del alcance acordado |
| UPSTREAM | Clientes, DNS, requisitos de SLO/RTO/RPO |
| DOWNSTREAM | ALB, Target Group, EC2, ASG, AZ, capa de datos |
| LATERALES | CloudWatch, VPC, IAM, health checks, backups |
| SCOPE | Instancia, AZ, región y dependencias externas |
| FAILURE BOUNDARY | Multi-AZ cubre dominio AZ, no automáticamente región; duplicación EC2 no replica estado |
| NO ES SU TRABAJO | HA no garantiza cero downtime; FT de un componente no garantiza FT del TODO; durabilidad no es disponibilidad |
| EXAM TRIGGER | no single point of failure; automatic replacement; multi-AZ; uptime vs data loss |

## Arquitectura LEGO
```text
DNS:     Route 53 ──resuelve──► ALB endpoint
TRÁFICO: Cliente ──────────────► ALB ─► Target Group ─► EC2 en AZ-A y AZ-B
CONTROL: Launch Template ─► ASG ─► mantener/reponer/escalar instancias
SALUD:   ALB/TG ─► health checks; ASG ─► salud/reemplazo configurado
ESTADO:  sesiones/DB/storage deben tener resiliencia propia
MÉTRICAS: CloudWatch observa; no es un salto del request
```

## Opciones descartadas: POR QUÉ
- Solo una EC2: SPOF.
- Solo varias EC2 en una AZ: una AZ sigue siendo SPOF.
- Solo ALB sin capacidad de reemplazo: distribuye lo existente, no repone.
- Solo ASG sin tráfico distribuido: repone instancias pero no decide destino de HTTP.
- Solo dos ENI/RAID1: resuelve fallos de componentes acotados, no disponibilidad end-to-end.
- Solo S3 de alta durabilidad: datos preservados no implican app disponible.

## Continuar leyendo
[[High Availability Fault Tolerance Durability - AWS Onion LEGO]]
[[EC2 Auto Scaling - Maintain Adjust Functional Onion]]
[[AWS-Onion - LEGO Architecture Map]]
