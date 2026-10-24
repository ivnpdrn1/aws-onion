---
type: service-dna
service: Elastic Load Balancing
project: AWS-Onion
updated: 2026-10-23
tags: [aws, onion, lego, elb, alb, nlb, gwlb, architecture]
---
# Elastic Load Balancing — DISTRIBUTE | Service DNA

## 1. Problema y esencia
Una instancia única es un punto de fallo y cuello de botella. Elastic Load Balancing expone un punto de entrada lógico y distribuye solicitudes o conexiones entre targets saludables. No crea servidores, no reemplaza instancias, no conserva por sí mismo las sesiones del usuario y no equivale a Auto Scaling.

**VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN**
- **ELB DISTRIBUTE** → tráfico → seleccionar destino saludable según listener/reglas y target groups.
- **Listener ACCEPT/ROUTE** → protocolo/puerto → recibir conexión y aplicar reglas (en ALB).
- **Target Group GROUP** → destinos registrados → define la agrupación, puerto/protocolo de destino y health check.
- **Health Check TEST** → respuesta del target → evaluar salud tras umbrales, no necesariamente prueba de todas las funciones de la app.
- **EC2/ECS/Lambda EXECUTE** → workload → procesar petición.
- **ASG MAINTAIN/ADJUST** → capacidad EC2 → reponer o escalar instancias. Para que use estado ELB deben configurarse los health checks del ASG apropiadamente; no se debe asumir una notificación directa e incondicional desde ALB.
- **CloudWatch OBSERVE** → métricas/alertas → informa decisiones operativas y scaling policies.
- **Multi-AZ DISTRIBUTE FAILURE DOMAINS** → riesgo de AZ → limita dependencia de una sola zona.

## 2. Contratos: ¿qué entra y sale?
| LEGO | Entrada | Decide o realiza | Entrega / dependencia |
|---|---|---|---|
| Route 53 | dominio | RESOLVE nombre | DNS del ELB (no mueve el tráfico HTTP) |
| ELB | conexión/solicitud | DISTRIBUTE | target saludable |
| Listener | puerto/protocolo | ACCEPT y, para ALB, reglas | acción/routing |
| Target Group | targets registrados | GROUP + health config | lista elegible |
| Health Check | respuesta HTTP/TCP/etc. según tipo | TEST | estado de salud |
| ASG | desired capacity, políticas, health | MAINTAIN/ADJUST | instancias nuevas/reemplazadas |
| Security Groups/NACL/Routes | reglas de red | ALLOW/DENY/ROUTE | conectividad LB → target |
| Application | request | EXECUTE | respuesta |

## 3. Recorrido ONION: datos ≠ control
**Data plane (recorrido conceptual, no diagrama físico literal):**
```text
Usuario → DNS(Route53: resolver) → ALB listener :443 → regla
       → Target Group → target saludable en AZ-A/AZ-B → aplicación
```
**Control/observabilidad (no son saltos del paquete):**
```text
Health checks → salud del target → ELB decide elegibilidad
ASG (health checks ELB habilitados + grace periods) → reemplazo EC2
Scaling policy ← métricas CloudWatch → ASG ajusta capacidad
```
Importante: un fallo no garantiza continuidad de una sesión existente. Las conexiones en curso pueden cortarse; reintentos, sesiones externas, aplicación stateless y almacenamiento compartido requieren diseño adicional.

## 4. Secuencia de fallo LEGO
1. EC2-A deja de responder al health check según thresholds configurados.
2. Target Group marca EC2-A como unhealthy.
3. ELB generalmente evita nuevos envíos a ese target si quedan targets saludables; el comportamiento sin targets saludables puede diferir según configuración (fail-open).
4. ASG puede reemplazar EC2-A **si** tiene ELB health checks habilitados y pasa su health grace period, o por sus propios EC2 status checks.
5. Launch Template/ASG crea EC2-D, bootstrap configura la aplicación, registro e health check validan; ELB vuelve a enrutar cuando esté healthy.
6. Si la app guarda sesiones localmente, falla la continuidad: externalizar estado y administrar reintentos.

## 5. Elegir tipo de ELB (y descartar alternativas)
| Requisito | Elegir | Razón / descarte |
|---|---|---|
| HTTP/HTTPS, rutas /orders vs /account, hostname api vs shop | ALB L7 | inspecciona información HTTP; NLB L4 no hace path-based routing |
| Web/microservicios/Lambda targets | ALB | reglas de aplicación, targets apropiados |
| TCP/UDP, baja latencia, IP estática por AZ, PrivateLink endpoint services | NLB L4 | conexión/red; ALB no ofrece el mismo contrato de IP estática nativa |
| Distribuir tráfico a firewalls virtuales/IDS/IPS | GWLB | appliances y GENEVE; ni ALB ni NLB reemplazan esta inspección |
| Classic Load Balancer | legacy | preferir ALB/NLB/GWLB para diseños nuevos según requisito |

Matiz: ALB puede hacer terminación HTTPS; NLB admite listeners TLS y TLS termination según caso. Static IP de NLB se refiere al frente del balanceador; no significa que instancias de backend tengan IP pública.

## 6. Dependencias y errores típicos
- Listener sin target group/rule válido: no hay backend.
- TG healthy pero SG impide entrada desde ALB: comprobar puertos/rutas y health check.
- Health path /health devuelve 200, pero base de datos caída: check insuficiente (superficial).
- 1 sola AZ: ELB no crea por sí solo Multi-AZ.
- ASG desired=1: reemplazo eventual, no disponibilidad simultánea ante fallo.
- Stickiness: no equivale a persistencia durable de sesión.
- ALB no crea instancias; ASG no enruta solicitudes.

## 7. EXAM CRAM
- **Route53 = WHERE; ELB = WHICH; TG = WHO/HEALTH; ASG = HOW MANY; EC2 = EXECUTE; CloudWatch = OBSERVE.**
- HTTP/HTTPS, host/path routing, Lambda → ALB L7.
- TCP/UDP, ultra-low latency, static IP/PrivateLink → NLB L4.
- Firewall fleet, inspection, GENEVE → GWLB.
- HA ≠ FT estricta; Multi-AZ reduce dominio de fallo pero no arregla SPOF de datos/sesiones.
- Health check de ELB protege **routing**; health check de ASG protege **capacidad** si está integrado correctamente.

## 8. Puentes y lectura
- [[EC2 Auto Scaling - Maintain Adjust Functional Onion]]
- [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]]
- [[High Availability Fault Tolerance Durability - AWS Onion LEGO]]
- [[EC2 User Data - Bootstrap LEGO Service DNA]]
- [[Elastic Load Balancing - Cuando la puerta tiene varios servidores]]
Fuente inicial: lección aportada por el usuario, «Amazon Elastic Load Balancing.txt» (2026-10-23). Las salvedades sobre sesiones, fail-open y activación de health checks ASG son ampliaciones arquitectónicas para evitar interpretaciones incorrectas.
