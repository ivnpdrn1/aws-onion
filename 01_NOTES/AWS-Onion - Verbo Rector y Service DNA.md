---
type: framework
domain: architecture
updated: 2026-10-16
tags: [aws-onion, architecture, verbs, relationships, certification, service-dna]
---
# AWS-ONION — Verbo rector, Service DNA y engranaje arquitectónico

> [!abstract] Principio rector
> **VERB → RESPONSIBILITY → RELATIONSHIP → DECISION → ARCHITECTURE**
> Memorizar primero el **verbo rector**, después el objeto sobre el cual actúa, su alcance, sus dependencias y qué **NO** hace.

## 1. No es un catálogo: es un grafo de responsabilidades
Cada servicio debe documentarse bajo el siguiente contrato:

| Campo | Pregunta |
|---|---|
| **VERB** | ¿Cuál es su acción principal? |
| **OBJECT** | ¿Sobre qué opera el verbo? |
| **WHY** | ¿Qué requisito resuelve? |
| **SCOPE** | ¿Es global, regional, AZ, VPC o recurso? |
| **UPSTREAM** | ¿Qué necesita para operar o qué origina la acción? |
| **DOWNSTREAM** | ¿A quién entrega el resultado? |
| **COOPERATES** | ¿Qué servicios participan lateralmente? |
| **PLANE** | ¿Data, control, observabilidad o descubrimiento? |
| **FAILURE** | ¿Qué fallo mitiga y cuál no? |
| **EXAM TRIGGER** | ¿Qué palabras del escenario lo hacen candidato? |
| **NOT ITS JOB** | ¿Qué opción similar descarto y por qué? |

## 2. Diccionario de verbos para Traffic & Capacity Onion
| Servicio/componente | Verbo rector | Objeto | Relación que hay que recordar | No confundir |
|---|---|---|---|---|
| Route 53 DNS | **RESOLVE / DIRECT** | Nombre → endpoint | Devuelve respuestas DNS que orientan hacia ALB regional u otros endpoints | DNS no transporta las peticiones HTTP |
| ALB / ELB | **DISTRIBUTE** | Solicitudes/conexiones → targets | Listener/reglas + target group + healthy targets | Un solo ELB no balancea entre regiones |
| Listener / rules | **MATCH / FORWARD** | Protocolo, puerto, host, path | Seleccionan una acción o Target Group | No crean instancias |
| Target Group | **GROUP / CHECK** | Targets registrados y estado de salud | ALB usa grupo para elegir targets; ASG registra/desregistra | No es un salto físico del paquete |
| EC2 | **EXECUTE** | Workload | Ejecuta la app que atiende el target | No escala automáticamente por sí solo |
| CloudWatch | **OBSERVE** | Métricas, logs, alarmas | Recoge métricas de EC2 y ALB utilizadas por políticas y alarmas | No aprovisiona instancias |
| Scaling policy | **EVALUATE / TARGET** | Señales y objetivos de capacidad | Transforma criterio de demanda en ajuste de desired capacity | Target tracking puede gestionar alarmas automáticamente |
| Auto Scaling Group | **MAINTAIN / ADJUST** | Cantidad de EC2 | Lanza/termina y opcionalmente registra EC2 en Target Group | No enruta requests |
| Route 53 health checks | **ASSESS** | Salud del endpoint/región | Contribuyen a DNS failover o routing condicionado por salud | No sustituyen health checks internos del ELB |
| CloudWatch alarm | **DETECT / SIGNAL** | Umbral/condición de métrica | Puede notificar o iniciar acciones compatibles | No es obligatoria para todo tipo de escalado |

## 3. Dos rutas diferentes, nunca mezclarlas
```text
DESCUBRIMIENTO Y TRÁFICO:
Cliente -- consulta DNS --> resolver / Route 53
Cliente <-- endpoint ------- DNS
Cliente ------------------> ALB regional --> target EC2 --> aplicación

OBSERVACIÓN / AJUSTE:
EC2 y ALB -- métricas --> CloudWatch
Política de scaling (métricas/objetivos/horarios) --> ASG
ASG --> crea/retira EC2 --> actualiza targets registrados
ELB health checks --> evalúa targets
```
El orden del gráfico es **lógico**, no implica que Route 53, Target Group, CloudWatch o ASG sean dispositivos que atraviese cada petición.

## 4. Vertical, Horizontal y Functional
- **Vertical = BETWEEN**: del nombre al endpoint, del ALB al workload; depende del alcance.
- **Horizontal = WITHIN**: Route 53 (descubrimiento), ALB (distribución), CloudWatch (observación), ASG (capacidad) cooperan para disponibilidad.
- **Functional = THROUGH TIME**: una solicitud fluye por el data plane; métricas y health checks pueden provocar decisiones posteriores en el control plane.

## 5. Service DNA — ejemplo ALB
| Campo | Valor |
|---|---|
| VERB | DISTRIBUTE |
| OBJECT | HTTP/HTTPS requests hacia targets configurados |
| WHY | Escalar tráfico y reducir dependencia de una instancia |
| SCOPE | Regional, puede abarcar múltiples AZ de la región |
| UPSTREAM | Cliente después de obtener endpoint DNS |
| DOWNSTREAM | Listener → Rule → Target Group → target |
| COOPERATES | Route 53, EC2, ASG, CloudWatch, Security Groups |
| PLANE | Data plane de solicitudes; configuración por control plane |
| FAILURE | Evita enviar normalmente a targets no saludables; requiere targets sanos y capacidad |
| EXAM TRIGGER | HTTP host/path routing, multiple EC2 targets |
| NOT ITS JOB | No agrega instancias ni balancea un único ALB sobre varias regiones |

## 6. Método para examen y diseño real
1. Subrayar el **verbo** escondido en el requisito (resolver, distribuir, observar, escalar, ejecutar).
2. Identificar el **objeto** (DNS name, HTTP request, metric, EC2 capacity).
3. Identificar **scope** (región vs AZ vs recurso).
4. Dibujar relaciones **upstream/downstream** y conexiones **laterales**.
5. Separar **data plane** del **control plane**.
6. Elegir el servicio cuyo verbo y alcance coincidan.
7. Descartar opciones cuyo verbo o alcance no coincidan.
8. Validar resiliencia, seguridad, costes y dependencias de datos.

## 7. Anclas de memoria
**Route 53 = WHERE. ALB = WHICH. CloudWatch = OBSERVE. ASG = HOW MANY. EC2 = EXECUTE.**

**El verbo es el gancho de memoria; las relaciones son el conocimiento arquitectónico.**

## Catálogo implementado

- [[AWS-Onion - LEGO Architecture Map]]
- Directorio `02_SERVICES/` con 18 fichas normalizadas; notas detalladas en `01_NOTES/`.
- Cada pieza lleva puertos de entrada/salida, alcance y límites; no confundir **relación lógica** con secuencia de paquetes.

## Relaciones
- [[Route 53 - DNS vs Load Balancing Multi-Region Onion]]
- [[ALB - Target Groups and Auto Scaling Functional Onion]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[AWS-Onion - Functional Onion]]
- [[AWS-Onion - Exam Map]]
