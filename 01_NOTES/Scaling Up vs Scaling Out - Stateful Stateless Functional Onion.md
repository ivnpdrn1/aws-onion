---
type: concept
domain: compute-architecture
updated: 2026-10-18
tags: [aws-onion, scaling-up, scaling-out, stateful, stateless, ec2, auto-scaling, alb, certification]
---
# Scaling Up vs Scaling Out — Stateful / Stateless Functional Onion

> [!abstract] Memoria rectora
> **ENLARGE** (Scaling Up) = ampliar recursos de una instancia.
> **REPLICATE** (Scaling Out) = aumentar instancias.
> **EXTERNALIZE** (state) = separar datos/sesiones persistentes del servidor de aplicación cuando sea necesario para replicarlo con seguridad.
> **DISTRIBUTE** (ALB) → **OBSERVE** (CloudWatch) → **ADJUST** (ASG).

## 1. Derivado de la clase adjunta
La transcripción **Scaling Up vs Scaling Out.txt** presenta:
- *Scaling Up / Vertical Scaling*: cambiar a un EC2 instance type de mayor CPU/RAM; a menudo requiere detener y reiniciar la instancia.
- *Scaling Out / Horizontal Scaling*: añadir varias instancias equivalentes y balancear solicitudes entre ellas.
- *Stateless*: cada servidor no necesita retener información de sesión local entre solicitudes.
- *Stateful*: existe información que debe recordarse, como compras; un sistema de ecommerce puede tener **web tier stateless** y **database stateful**.
- Para escalar web tiers dinámicos, trasladar archivos compartidos a EFS y datos a una base externa como RDS o DynamoDB puede hacer a las instancias reemplazables.
- Escalar horizontalmente reparte carga y reduce la dependencia de una instancia, con posibilidad de colocar instancias en diferentes AZ; ciertas cargas como componentes de escritura de base de datos requieren a veces escalar verticalmente.

**Nota:** La clase explica estos conceptos a alto nivel. El diseño más detallado a continuación es una *ampliación arquitectónica*, no transcripción literal.

## 2. Contrato LEGO
| Concepto | VERBO | OBJETO | RESPONSABILIDAD | RELACIONES | DECISIÓN |
|---|---|---|---|---|---|
| Vertical scaling | **ENLARGE** | Recursos de una instancia | Más potencia para una unidad de ejecución | EC2 instance type; límites/reinicio | Proceso no distribuible; CPU/RAM por host |
| Horizontal scaling | **REPLICATE** | Número de instancias | Añadir réplicas para repartir trabajo | ASG, EC2, ALB, Target Group | Solicitudes/trabajo distribuible |
| Stateless web tier | **DECOUPLE** | Estado de la instancia | Mantener servidores reemplazables | Datos/sesiones externos | Evitar afinidad no deseada |
| External state | **EXTERNALIZE** | Sesiones, archivos, datos | Persistencia fuera de cómputo efímero | EFS, RDS, DynamoDB (según tipo) | Disponibilidad y consistencia |
| ALB | **DISTRIBUTE** | Solicitudes | Balancear sobre targets | Target Group, health checks | Elegir destino elegible |
| CloudWatch | **OBSERVE** | Métricas | Proveer señales de utilización | EC2, ALB, políticas ASG | ¿Cuándo medir demanda? |
| ASG | **ADJUST** | Capacidad EC2 | Lanzar/retirar instancias | Scaling policy, Launch Template, Target Group | ¿Cuántas instancias? |

## 3. Flujo de solicitudes versus control de capacidad
```text
DISCOVERY: Cliente --DNS query--> Route 53 --DNS answer--> cliente
DATA PLANE: Cliente -------------------> ALB -> (reglas/TG) -> EC2 stateless -> app
                                                            |
                                         state/data -------> EFS / RDS / DynamoDB
OBSERVATION: ALB/EC2 --metrics--> CloudWatch
CONTROL: scaling policy --desired capacity--> ASG --launch/terminate--> EC2
                                    ASG --register/deregister--> Target Group
```
**No** colocar Route 53, CloudWatch, ASG o Target Group como si todos fueran saltos físicos consecutivos en el recorrido HTTP. EFS, RDS y DynamoDB son **alternativas o combinaciones justificadas**, no servicios obligatorios simultáneos.

## 4. ¿Por qué importa el estado?
- Si el carrito de compras solo reside en memoria de EC2-A, una solicitud posterior atendida en EC2-B no puede recuperar ese carrito automáticamente.
- El **sistema** puede ser stateful, mientras que el **web/application tier** es stateless.
- Diseño: identificar cuál dato debe persistir, quién lo lee/escribe y con qué requisitos de latencia/consistencia; externalizar el estado que impida reemplazar instancias.
- **Escalar app tier no escala automáticamente la base de datos.** Los cuellos de botella se desplazan.
- Sticky sessions no equivalen a persistencia durable ni resuelven pérdida de servidor.

## 5. Cuándo elegir
| Requisito | Elección | Qué descartar |
|---|---|---|
| Un proceso indivisible necesita más RAM/CPU | **Scaling Up** | Agregar EC2 no agranda memoria de un único proceso |
| Muchas solicitudes web independientes | **Scaling Out** + ALB + estado desacoplado | Una sola instancia grande como única estrategia resiliente |
| Recuperar sesión en distintas EC2 | **Externalize state** | Mantener sesión solo en RAM local |
| Necesidad de archivos compartidos | **EFS** donde encaja filesystem | EBS de instancia aislada como filesystem compartido genérico |
| Datos SQL transaccionales | **RDS** cuando corresponde | EFS como sustituto de DB relacional |
| Datos NoSQL | **DynamoDB** cuando corresponde | Asumir SQL/RDS sin evaluar modelo |
| Resistencia a fallo de AZ | ASG y ALB multi-AZ, dependencia de datos resiliente | Solo Scaling Up |

## 6. Reglas de examen y producción
1. Subrayar palabras: **single process more memory** → ENLARGE; **more web requests / add instances** → REPLICATE.
2. Antes de replicar: ¿**stateful** o **stateless**? ¿Dónde se preservan sesiones y datos?
3. Separar quién **DISTRIBUTES** (ALB), quién **OBSERVES** (CloudWatch) y quién **ADJUSTS** (ASG).
4. Scaling Out permite diseñar tolerancia a fallos de instancia/AZ si se distribuye capacidad; no garantiza por sí mismo alta disponibilidad.
5. Scaling vertical puede exigir una interrupción; revisar el tipo de recurso, límites y estrategia de cambio.
6. Pensar en el sistema completo: cómputo, almacenamiento, datos, red, seguridad, costos y cuellos de botella.

## 7. Autoevaluación
- ¿Por qué 8 EC2 pequeñas no equivalen necesariamente a una EC2 grande?
- ¿Qué estado rompe la posibilidad de sustituir servidores libremente?
- ¿Quién añade EC2 y quién balancea requests?
- ¿Escalar seis EC2 resuelve por sí solo un cuello de botella de escritura en la DB?
- ¿Cómo distinguirías disponibilidad de capacidad?

## Relaciones Obsidian
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
- [[ALB - Target Groups and Auto Scaling Functional Onion]]
- [[EC2 - Architecture Patterns and Auto Scaling]]
- [[CloudWatch - Mental Model]]
- [[EC2]]
- [[Auto Scaling Group]]
- [[ALB]]
- [[CloudWatch]]
- [[Amazon EFS]]
- [[Amazon RDS]]
- [[Amazon DynamoDB]]
