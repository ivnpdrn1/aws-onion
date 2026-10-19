---
type: service-dna
service: Amazon DynamoDB
verb: STORE
object: NoSQL key-value/document data
updated: 2026-10-18
tags: [aws-onion, service-dna, database, nosql]
---
# Amazon DynamoDB — STORE
| Contrato | Definición |
|---|---|
| VERBO | **STORE** |
| OBJETO | Datos NoSQL clave-valor y documento |
| RESPONSABILIDAD | Almacenar datos administrados con opciones de capacidad y acceso por claves/índices |
| SCOPE | Regional con replicación multi-AZ administrada |
| UPSTREAM | Aplicación / SDK / API con permisos IAM |
| DOWNSTREAM | Lectura/escritura de ítems persistentes |
| COOPERATES | EC2, IAM, autoscaling propio de capacidad donde aplique |
| NOT ITS JOB | Sustituir indistintamente una base SQL relacional o escalar EC2 |
| EXAM TRIGGER | Managed scalable NoSQL key-value/document |

**Relación LEGO:** [[EC2]] EXECUTE → [[Amazon DynamoDB]] STORE. Ver [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]].
