---
type: concept
domain: compute-architecture
updated: 2026-10-20
tags: [aws-onion, ec2, auto-scaling, launch-template, cloudwatch, elb, health-checks, availability, certification]
---
# EC2 Auto Scaling — MAINTAIN + ADJUST (Functional Onion)

> [!abstract] Verbo y objeto
> **MAINTAIN / ADJUST → EC2 fleet capacity**. El ASG sostiene la capacidad deseada y la modifica según una política o una intervención. No recibe ni balancea las solicitudes del usuario.

## Fuente de clase
Basado en **Amazon EC2 Auto Scaling (2).txt** (transcripción provista). La clase presenta:
- Reemplazo de instancias defectuosas (**maintain availability**) y crecimiento/reducción en respuesta a demanda (**scale out/in**).
- Relaciones: EC2, CloudWatch metrics/alarms, ELB, VPC/subnets/AZ, opciones On-Demand/Spot y capacidad para nodos de contenedores.
- Launch Template (AMI, tipo de EC2, EBS, instance profile/IAM), ASG con tamaño y políticas, health checks EC2 y ELB, grace period.
- Modalidades **manual**, **dynamic**, **predictive** y **scheduled**.

Las precisiones de arquitectura a continuación amplían la clase y se distinguen de su contenido textual.

## Contrato LEGO normalizado
| Campo | Respuesta |
|---|---|
| **VERBO** | **MAINTAIN / ADJUST** |
| **OBJETO** | Cantidad y salud de las instancias EC2 administradas por un ASG |
| **RESPONSABILIDAD** | Mantener la capacidad deseada y escalar dentro de los límites del grupo |
| **SCOPE** | ASG regional; puede incluir subnets de varias AZ |
| **UPSTREAM** | Launch Template, configuración min/desired/max, salud de EC2/ELB, políticas y programación |
| **DOWNSTREAM** | Lanzar/terminar EC2, registrar/desregistrar targets con Target Groups asociados |
| **COOPERATES** | [[EC2]], [[ALB]], [[Target Group]], [[CloudWatch]], [[VPC]], [[Subnet]], [[Amazon EFS]], [[Amazon RDS]], [[Amazon DynamoDB]] |
| **NOT ITS JOB** | No balancea solicitudes, no replica datos de la aplicación, no agranda verticalmente la memoria de cada instancia |
| **EXAM TRIGGER** | Failed instances replaced automatically; scale out/in; scheduled/predictive capacity; desired/min/max |

## Tres piezas distintas
1. **Launch Template — DEFINE:** cómo crear una EC2 (AMI, instance type, storage EBS, IAM instance profile, SG, user data). No decide el número.
2. **Auto Scaling Group — MAINTAIN:** grupo regional, AZ/subnets, min/desired/max y sustitución de instancias defectuosas.
3. **Scaling Policy — ADJUST CRITERIA:** cuándo y cómo modificar desired capacity; puede depender de métricas, objetivo o programa según modalidad.

## Relación vertical y horizontal
```text
DISCOVERY: Client -> DNS resolution / Route 53 -> ALB endpoint
TRAFFIC:   Client -------------------------------> ALB -> Target Group selection -> EC2 -> App

OBSERVE:   EC2 / ALB -- metrics ------------------> CloudWatch
DECIDE:    metrics + scaling policy (or schedule) -> scaling action
ADJUST:    ASG -> launches/terminates EC2 -> targets registered/deregistered
HEALTH:    EC2 status + optionally ELB checks -> ASG replacement decision
PLACE:     VPC / subnets / Availability Zones -> permitted placement
```
**Son planos diferentes**: CloudWatch, ASG y Target Group no son dispositivos intermedios del paquete HTTP; Route 53 resuelve DNS, no transporta cada request.

## Health checks — dos preguntas
- **EC2 status checks:** ¿funciona la instancia desde la perspectiva de los controles EC2?
- **ELB health checks:** ¿está preparada la aplicación/target para responder la comprobación configurada?
- **ASG:** evalúa salud y reemplaza instancias según las comprobaciones habilitadas; la integración de salud ELB con ASG debe estar correctamente configurada.
- **Health check grace period:** concede tiempo de inicialización antes de que ASG actúe sobre determinadas evaluaciones de salud de instancias recientes; no significa que el ALB deba servir tráfico a un target aún unhealthy.

## Modos y verbos de examen
| Modalidad | VERBO | Disparador |
|---|---|---|
| Manual | SET | Un operador cambia la capacidad |
| Dynamic | REACT | Métrica/objetivo de demanda |
| Predictive | ANTICIPATE | Pronóstico basado en patrones históricos |
| Scheduled | SCHEDULE | Hora/fecha prevista |

Ejemplo de parámetros: **min=2, desired=4, max=8**. Desired es objetivo, no garantía de que siempre se observen exactamente cuatro instancias durante eventos de convergencia.
El ejemplo de CPU por encima de 80% de la clase es ilustrativo, no un umbral predeterminado obligatorio.

## Relaciones con Scaling Up / Out y State
- **Scaling Up — ENLARGE:** se cambia potencia de una instancia; un ASG EC2 no lo realiza por el simple hecho de agregar nodos.
- **Scaling Out — REPLICATE:** ASG lanza más instancias; ALB reparte nuevas peticiones entre targets elegibles.
- **Stateful / Stateless — EXTERNALIZE:** para sustituir servidores sin perder sesiones/estado duradero, identificar y externalizar datos donde corresponda. EFS comparte archivos, RDS administra SQL, DynamoDB guarda datos NoSQL; no se requieren los tres simultáneamente.
- **Disponibilidad ≠ capacidad:** reemplazo por fallo y cambio por demanda son decisiones independientes; distribuir entre AZ mejora resiliencia si las otras dependencias también lo permiten.
- **Coste:** escala hacia dentro y opciones Spot/On-Demand según tolerancia a interrupciones y objetivos.

## Ejemplo examen: escoger y descartar
| Requisito | Elegir | Descartar |
|---|---|---|
| Reemplazar EC2 fallida sin que aumente el tráfico | ASG health replacement | CloudWatch solo: observa, no lanza |
| EC2 nuevas con configuración idéntica | Launch Template + ASG | ALB: distribuye, no aprovisiona |
| Incrementar capacidad cuando sube el tráfico | Dynamic scaling + ASG | Solo Route 53: resuelve DNS |
| Escalar cada lunes antes del pico | Scheduled scaling | Depender únicamente de respuesta reactiva |
| Fallo en endpoint web aunque instancia está encendida | ELB target health + habilitar su evaluación en ASG | Solo status checks EC2 |
| Mantener usuarios al cambiar de instancia | Externalizar estado de app | Sticky sessions como sustituto de persistencia |

## Secuencia para diseñar
**Requirement → DEFINE instance → PLACE (VPC/AZ) → MAINTAIN capacity → VERIFY health → OBSERVE demand → ADJUST capacity → DISTRIBUTE traffic.**

Es una **secuencia de decisiones y operación**, no una cadena física por la que pasan los paquetes.

## Links
- [[Auto Scaling Group]]
- [[EC2]]
- [[ALB]]
- [[Target Group]]
- [[CloudWatch]]
- [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Exam Map]]
