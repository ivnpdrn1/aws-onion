---
type: concept
domain: reliability-architecture
updated: 2026-10-23
tags: [aws-onion, lego, high-availability, fault-tolerance, durability, multi-az, architecture, certification, book]
---
# High Availability, Fault Tolerance y Durability — AWS-ONION-LEGO

> [!abstract] Principio rector
> **Asumir el fallo de una pieza; diseñar para que el TODO mantenga su función.** Identificar y eliminar *single points of failure* según el nivel de resiliencia requerido, sin confundir disponibilidad, continuidad perfecta ni preservación de datos.

## Fuente y límites
Fuente base: transcripción de clase **High Availability and Fault Tolerance.txt**, compartida el 2026-10-23. Enseña HA como interrupción mínima y FT como ausencia de interrupción, los ejemplos ELB, ASG, Route 53, Multi-AZ, redundancia de NIC/discos/alimentación y diferencia entre durabilidad y disponibilidad. Las precisiones marcadas como *ampliación arquitectónica* no deben atribuirse literalmente a la clase.

## VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN
| Pieza LEGO | Verbo → Objeto | Responsabilidad | No hace |
|---|---|---|---|
| Route 53 | RESOLVE/DIRECT → DNS/endpoints | Responder consultas DNS, políticas y failover DNS según configuración | No distribuye cada petición HTTP ni sustituye el balanceador |
| ALB/ELB | DISTRIBUTE → solicitudes/targets | Encaminar a targets saludables según listeners y reglas | No determina cantidad de EC2 |
| Target Group | GROUP/CHECK → targets | Agrupar y comprobar salud del destino | No crea instancias |
| EC2 | EXECUTE → aplicación | Ejecutar web/app (Apache, Nginx, otros) | No garantiza HA por sí misma |
| Auto Scaling Group | MAINTAIN/ADJUST → flota EC2 | Mantener desired capacity, reponer instancias y escalar | No balancea el tráfico ni replica datos |
| Availability Zone | ISOLATE → dominio de fallo | Separar despliegue físico para evitar una sola AZ | No garantiza disponibilidad de todas las capas de aplicación |
| CloudWatch | OBSERVE → métricas/alarms | Visibilidad y señales para políticas cuando se configuran | No reemplaza EC2 directamente |
| S3 | PRESERVE → objetos | Diseñado para alta durabilidad; disponibilidad es otra métrica | Durabilidad no significa acceso sin interrupciones |

## Mapa LEGO: planos distintos, no una única ruta física
```text
DISCOVERY: usuario --DNS query--> Route 53 --DNS answer--> ALB endpoint
DATA:      usuario --------------HTTPS----------------> ALB -> TG -> EC2/app
HEALTH:    ALB/TG --health checks-----------------------> targets
CONTROL:   ASG --maintain/replace/adjust---------------> EC2 + TG registration
OBSERVE:   métricas EC2/ALB ----------------------------> CloudWatch
PLACEMENT: VPC/subnets en AZ-A y AZ-B delimitan dónde vive cada EC2
DATA:      DB, sesiones y ficheros requieren su propio diseño resiliente
```

## Secuencia funcional — un servidor falla
1. EC2-B deja de estar saludable.
2. El ALB deja de elegirlo cuando la comprobación de salud lo detecta (pueden fallar solicitudes en curso).
3. EC2-A/C/D siguen atendiendo si tienen capacidad.
4. ASG repone una instancia si la considera no saludable, de acuerdo con la configuración de health checks.
5. La nueva instancia completa bootstrap, supera health checks y recibe tráfico.
6. Se valida la recuperación, sin confundir *desired* con cantidad instantánea exacta.

## Secuencia funcional — una AZ falla
- Con recursos repartidos en AZ-A/AZ-B, el tráfico puede concentrarse en AZ-B si las dependencias y la capacidad superviviente lo permiten.
- El ASG puede buscar reposición; no se garantiza aprovisionamiento instantáneo ni que pueda recuperar capacidad si hay problemas regionales/capacidad.
- El diseño de base de datos, almacenamiento, sesión y conectividad debe ser también Multi-AZ o apropiadamente redundante; **EC2 Multi-AZ por sí sola no hace toda la aplicación HA**.

## Distinciones esenciales
| Concepto | Pregunta | Objetivo | Matiz |
|---|---|---|---|
| High Availability (HA) | ¿Cuánto tiempo está accesible el servicio? | Minimizar interrupciones | Puede haber breve downtime |
| Fault Tolerance (FT) | ¿Puede un fallo individual ocurrir sin interrumpir servicio? | Continuidad ante fallos definidos | Requiere redundancia y mecanismos de transferencia; objetivo cero interrupción para fallos cubiertos, no inmunidad universal |
| Durability | ¿Sobrevive el dato sin pérdida/corrupción? | Preservación del dato | No garantiza acceso inmediato |
| Availability del dato | ¿Puedo leer/escribir ahora? | Accesibilidad | Un dato conservado podría no estar disponible |
| RTO | ¿Cuánto tiempo máximo de recuperación toleramos? | Recuperación | Objetivo de negocio, no valor universal |
| RPO | ¿Cuánta pérdida temporal de datos toleramos? | Recuperación de datos | Depende del mecanismo de replicación |

### Replicación: precisión técnica
- **Síncrona:** confirma escritura en los destinos requeridos antes de declarar una operación completada; reduce riesgo de pérdida de cambios confirmados, con coste en latencia y disponibilidad bajo particiones.
- **Asíncrona:** permite latencia menor y riesgo de perder cambios aún no replicados durante un fallo.
- **Ampliación:** la replicación síncrona por sí sola **no garantiza FT completa**, y FT no se reduce a RAID 1 o dos ENI. La continuidad extremo a extremo exige cubrir capa de aplicación, red, estado, almacenamiento y fallos de causa común.
- La clase usa RAID 1/NICs redundantes como ilustración conceptual; añadir otra ENI a una EC2 no convierte automáticamente la aplicación en fault-tolerant. EBS ya integra mecanismos de redundancia en su propio ámbito, pero no equivale a Multi-AZ para un volumen EBS individual.

### S3: los once nueves
S3 Standard está diseñado para **99.999999999% de durabilidad anual de objetos**, distinto de la disponibilidad del servicio. No confundir probabilidad estadística de pérdida con SLA de acceso, ni interpretar la cifra como garantía de cero pérdidas. La clase da el ejemplo de 10 millones de objetos y uno esperado cada 10.000 años.

## Pensamiento arquitectónico / examen
**Requirement → Failure domain → Verb → Object → Scope → Interfaces → State → Trade-off → Validation.**
- “Distribuir tráfico entre instancias saludables” → ELB.
- “Reponer instancias y conservar desired” → ASG.
- “Sobrevivir a una AZ” → Multi-AZ + redundancia de **todas** las dependencias críticas.
- “Failover de endpoints/regiones por DNS” → Route 53 routing/health checks, considerando caché/TTL.
- “Proteger datos contra pérdida” → durabilidad, backups, replicación/controles de datos; no confundir con uptime.
- “Cero pérdida de datos confirmados” → discutir garantías de escritura, consistencia, mecanismos de replicación, RPO.
- “Sin interrupción observable” → tolerancia a fallos para modos de fallo definidos; justificar complejidad y coste.

## Experimento mental LEGO
1. Retira una EC2: ¿quién detecta, deja de enviar tráfico y repone?
2. Retira una AZ: ¿qué piezas sobreviven y qué dependencias se vuelven SPOF?
3. Retira el almacenamiento o el servicio de identidad: ¿sigue funcionando el TODO?
4. Cambia Apache por Nginx: ¿cambia el objetivo de resiliencia o solamente la pieza SERVE?
5. Cambia un requisito de 99,9% a 99,99%: ¿qué impacto tienen dependencias, presupuesto de interrupción y precio?
6. Evalúa *qué falla*, *qué percibe el usuario* y *qué reconstruye el sistema*.

## Relaciones a otros LEGO
[[AWS-Onion - LEGO Architecture Map]] · [[EC2 Auto Scaling - Maintain Adjust Functional Onion]] · [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]] · [[Route 53 - DNS vs Load Balancing Multi-Region Onion]] · [[EC2 User Data - Bootstrap Apache IMDSv2 LEGO Onion]]

## Frase para el futuro libro
> **La fortaleza de AWS no está en una pieza que nunca falla: está en cómo elegimos y conectamos piezas para que el TODO sobreviva a fallos previsibles.**
