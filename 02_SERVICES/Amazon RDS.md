---
type: service-dna
service: Amazon RDS
verb: PERSIST
object: Relational data
updated: 2026-10-18
tags: [aws-onion, service-dna, database, relational]
---
# Amazon RDS — PERSIST
| Contrato | Definición |
|---|---|
| VERBO | **PERSIST** |
| OBJETO | Datos relacionales |
| RESPONSABILIDAD | Administrar motores relacionales compatibles con sus opciones de almacenamiento, backups, disponibilidad y escalado |
| SCOPE | Regional; recursos/instancias con colocación por AZ |
| UPSTREAM | Aplicación/EC2 y sus consultas SQL |
| DOWNSTREAM | Datos transaccionales persistentes |
| COOPERATES | EC2, VPC, SG, backups, réplicas según motor |
| NOT ITS JOB | Escalar las EC2 del web tier ni repartir solicitudes HTTP |
| EXAM TRIGGER | Managed relational database; SQL transactions |

**Relación LEGO:** [[EC2]] EXECUTE → [[Amazon RDS]] PERSIST. Ver [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]].
