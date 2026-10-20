---
type: architecture-map
updated: 2026-10-16
tags: [aws-onion, lego, service-dna, architecture, index]
---
# AWS-Onion — LEGO Architecture Map

> [!abstract] Reglas
> **VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN ARQUITECTÓNICA**
>
> **Un orden lógico por objetivo, NO una secuencia física universal.** Cada pieza tiene interfaces. Un recurso puede enlazarse con muchos otros; un servicio global no está necesariamente en el data plane y un componente de observabilidad tampoco es un salto de cada request.

## 1. Construcción por planos — piezas reutilizables
| Plano | Pregunta | Piezas LEGO (verbo) |
|---|---|---|
| Descubrimiento | ¿Dónde encontrar el endpoint? | [[Route 53]] RESOLVE |
| Tráfico | ¿Quién recibe y distribuye? | [[ALB]] DISTRIBUTE; [[Target Group]] GROUP |
| Compute | ¿Quién ejecuta? | [[EC2]] EXECUTE; [[Nitro System]] OFFLOAD |
| Capacidad | ¿Cuántos recursos? | [[Auto Scaling Group]] ADJUST |
| Observabilidad | ¿Qué ocurre? | [[CloudWatch]] OBSERVE |
| Red | ¿Dónde existe y cómo llega la comunicación? | [[VPC]] ISOLATE; [[Subnet]] SEGMENT; [[Route Table]] ROUTE; [[Internet Gateway]] CONNECT; [[NAT Gateway]] TRANSLATE; [[NACL]] FILTER; [[Security Group]] AUTHORIZE; [[ENI]] ATTACH |
| Identidad | ¿Quién puede hacer qué? | [[IAM Role]] DELEGATE; [[STS]] ISSUE |
| Operaciones | ¿Quién gestiona y diagnostica? | [[Systems Manager]] MANAGE |

## 2. LEGO A — web escalable en una región
```text
DNS discovery: User ⇢ Route 53 (RESOLVE) ⇢ ALB DNS endpoint
HTTP data:     User ────────────────────────► ALB (DISTRIBUTE)
                                                   │
                         Listener rules → Target Group (GROUP)
                                                   │
                                             EC2 (EXECUTE)
Observability: EC2 / ALB ─ metrics ─► CloudWatch (OBSERVE)
Capacity:     scaling policy ─► ASG (ADJUST) ─► EC2 and TG registrations
Networking:   VPC / subnet / routes / SG / ENI (required constraints)
```
**Route 53, Target Group, CloudWatch y ASG no son cuatro saltos físicos seriales.**

## 3. LEGO B — alta disponibilidad multi-region
```text
                      Route 53 (RESOLVE / DIRECT)
                     /                          \
             Region A endpoint            Region B endpoint
                   ALB                           ALB
               Target Group                   Target Group
                EC2 / ASG                      EC2 / ASG
```
Elegir failover Active-Passive o routing compatible con Active-Active; diseñar replicación y consistencia de datos, estado de sesión, RTO/RPO y límites de DNS TTL/caché. **Un ALB no distribuye targets cross-region.**

## 4. LEGO C — networking público/privado
```text
VPC (ISOLATE)
 ├── Public subnet (SEGMENT) → route 0.0.0.0/0 → IGW (CONNECT)
 │    └── Internet-facing ALB
 └── Private subnet (SEGMENT) → route 0.0.0.0/0 → NAT GW (TRANSLATE) [salida IPv4, si hace falta]
      └── EC2 (EXECUTE) + ENI (ATTACH) + SG (AUTHORIZE)
NACL (FILTER) afecta el límite subnet; Route Table (ROUTE) determina siguiente salto.
```
No se requieren NAT ni IGW en toda solución; seleccionar solo los bloques necesarios.

## 5. Reglas de montaje
1. **Comienza por el requisito**: disponibilidad, latencia, seguridad, coste, crecimiento, operación.
2. **Escoge VERB + OBJECT**, nunca nombres AWS al azar.
3. **Comprueba SCOPE** global/región/AZ/VPC/recurso para evitar conexiones imposibles.
4. **Une puertos**: salida de una pieza compatible con entrada de la otra.
5. **Dibuja los planos por separado**: discovery, data, control, observation y security.
6. **Evalúa fallo por nivel**: instancia, AZ, región; una protección no sustituye otra.
7. **Elimina piezas sin justificación**; preferir la mínima arquitectura que satisface requisitos.
8. **Verifica operabilidad y costos** antes de implementación.

## 6. Plantilla para nuevas piezas
```yaml
type: service-dna
service: <AWS service>
verb: <VERB>
object: <WHAT>
scope: <global|region|AZ|VPC|resource>
upstream: <inputs>
downstream: <outputs>
cooperates: <lateral relations>
failure_boundary: <what it can/cannot mitigate>
exam_trigger: <exam requirements>
not_its_job: <common wrong choice>
```

## 7. Índice del catálogo
[[Route 53]] · [[ALB]] · [[Target Group]] · [[EC2]] · [[CloudWatch]] · [[Auto Scaling Group]] · [[VPC]] · [[Subnet]] · [[Route Table]] · [[Internet Gateway]] · [[NAT Gateway]] · [[NACL]] · [[Security Group]] · [[ENI]] · [[IAM Role]] · [[STS]] · [[Systems Manager]] · [[Nitro System]]

## 8. Material anterior preservado
- [[AWS-Onion - Verbo Rector y Service DNA]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[AWS-Onion - Functional Onion]]
- [[Route 53 - DNS vs Load Balancing Multi-Region Onion]]
- [[ALB - Target Groups and Auto Scaling Functional Onion]]

## 9. LEGO D — Scaling Up vs Scaling Out (Stateful / Stateless)

**ENLARGE** (larger instance) vs **REPLICATE** (more instances). To scale a replaceable web tier, **EXTERNALIZE** the necessary state via [[Amazon EFS]] (SHARE), [[Amazon RDS]] (PERSIST), or [[Amazon DynamoDB]] (STORE), selected by data model, not installed by default.

Traffic: Route 53 DNS resolution → ALB DISTRIBUTE → EC2 EXECUTE.
Control: CloudWatch OBSERVE → scaling policy → ASG ADJUST → EC2 registration with Target Group.

Study: [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]].

## EC2 Auto Scaling — relación integrada
- [[EC2 Auto Scaling - Maintain Adjust Functional Onion]]
**Launch Template DEFINE → ASG MAINTAIN → CloudWatch OBSERVE / Scaling Policy DEFINE RESPONSE → ASG ADJUST → EC2 EXECUTE.** ELB DISTRIBUTE; Target Group GROUP/CHECK. Los health checks EC2 y ELB tienen alcances distintos. Consultar la nota para las modalidades manual, dynamic, predictive y scheduled y las distinciones Scaling Up/Out y stateful/stateless.
