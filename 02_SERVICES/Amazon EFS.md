---
type: service-dna
service: Amazon EFS
verb: SHARE
object: Shared elastic file system
updated: 2026-10-18
tags: [aws-onion, service-dna, state, filesystem]
---
# Amazon EFS — SHARE
| Contrato | Definición |
|---|---|
| VERBO | **SHARE** |
| OBJETO | Sistema de archivos compartido |
| RESPONSABILIDAD | Permitir que varias instancias utilicen archivos compartidos mediante NFS, con configuración de red/permisos |
| SCOPE | Servicio regional con mount targets dentro de la VPC/AZ correspondiente |
| UPSTREAM | EC2 o clientes compatibles que montan el sistema de archivos |
| DOWNSTREAM | Acceso a archivos persistentes compartidos |
| COOPERATES | EC2, VPC, Security Group, IAM según configuración |
| NOT ITS JOB | No es una base de datos relacional ni almacén genérico de sesiones |
| EXAM TRIGGER | Shared file access for many EC2 instances |

**Relación LEGO:** [[EC2]] EXECUTE → [[Amazon EFS]] SHARE. Ver [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]].
