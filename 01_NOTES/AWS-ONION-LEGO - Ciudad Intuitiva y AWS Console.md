---
type: aws-onion-lego-learning
project: AWS-Onion
updated: 2026-10-24
tags: [aws-onion, lego, intuitive-learning, vpc, alb, ec2, aws-console, obsidian]
aliases: [AWS ONION LEGO Ciudad, Arquitectura intuitiva AWS Console]
---
# AWS-ONION-LEGO — La ciudad, la recepción y los trabajadores

> [!abstract] Método permanente
> **NECESIDAD → ANALOGÍA → VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → AWS CONSOLE → DECISIÓN ARQUITECTÓNICA.**
> Cada pieza resuelve una necesidad concreta; reconstruir las relaciones antes de memorizar nombres. **El orden pedagógico no equivale a saltos físicos del paquete ni al orden de aprovisionamiento.**

## La historia en cuatro actos

1. **Encontrar** — Un cliente pide `example.com`; [[Route 53]] RESOLVE el nombre al DNS del ALB. DNS no transporta el HTTP.
2. **Entrar** — La [[VPC]] es la ciudad, el [[Internet Gateway]] su conexión exterior, las [[Route Table|Route Tables]] sus rutas, [[NACL]] el control del barrio y [[Security Group]] el guardia de recepción. El ALB público se ubica en subnets públicas de al menos dos AZ.
3. **Ser atendido** — [[ALB]] DISTRIBUTE; **Listener** LISTEN en HTTPS 443; **Listener Rule** DECIDE por host/ruta; [[Target Group]] GROUP y CHECK los targets; son funciones lógicas del balanceador, no edificios físicos consecutivos.
4. **Ejecutar** — rutas VPC locales y filtros de la subnet privada posibilitan ALB → SG de EC2 → [[ENI]]/IP privada → [[EC2]] EXECUTE. EC2 no necesita IP pública para ser target del ALB.

## Tabla maestra — pieza, esencia y consola

| Capa / servicio | Historia natural → VERBO / responsabilidad | AWS Console → Setting |
|---|---|---|
| 1. **Cliente** | **REQUEST** — Persona solicita una página | Fuera de AWS: navegador o cliente HTTP |
| 2. **Route 53** | **RESOLVE** — Guía de direcciones: nombre → DNS del ALB | Route 53 > Hosted zones > dominio > Create record (A Alias al ALB) |
| 3. **ALB DNS / IP públicas administradas** | **ADDRESS** — Dirección pública de la recepción; el ALB puede cambiar IP | EC2 > Load Balancers > ALB > Details > DNS name |
| 4. **VPC** | **ISOLATE** — Ciudad: define espacio de red y CIDR | VPC > Your VPCs > Create VPC |
| 5. **IGW** | **CONNECT** — Puerta de comunicación con Internet | VPC > Internet gateways > Create > Attach to VPC |
| 6. **Public Subnet** | **PLACE** — Barrio de recepción; subnets en dos AZ | VPC > Subnets > Create subnet |
| 7. **Public Route Table** | **ROUTE** — Señales hacia IGW: 0.0.0.0/0 → igw; asociar subnets | VPC > Route tables > Routes / Subnet associations |
| 8. **Public NACL** | **FILTER SUBNET** — Control de barrio público, sin estado; permitir retornos | VPC > Network ACLs > Inbound / Outbound / Subnet associations |
| 9. **ALB Security Group** | **FILTER RESOURCE** — Guardia de recepción: HTTPS 443 desde clientes; salida hacia targets | EC2 > Security Groups > Inbound / Outbound rules |
| 10. **ALB (ELB family)** | **DISTRIBUTE** — Recepción: distribuye solicitudes según listeners y reglas | EC2 > Load Balancers > Create > Application Load Balancer (internet-facing, dos AZ) |
| 11. **Listener HTTPS** | **LISTEN** — Ventanilla 443; certificado TLS y acción predeterminada | EC2 > Load Balancers > ALB > Listeners and rules > Add listener; ACM para certificado |
| 12. **Listener Rule** | **DECIDE / FORWARD** — Instrucción de recepción: ruta /api/* → target group | EC2 > Load Balancers > ALB > Listeners and rules > Rules |
| 13. **Target Group** | **GROUP / CHECK** — Directorio de trabajadores y Health Checks; no es salto físico | EC2 > Target Groups > Create target group > Targets / Health checks |
| 14. **VPC Internal Routing** | **ROUTE LOCAL** — Calles internas ALB → dirección privada del target (ruta local VPC) | VPC > Route tables > Routes (local) |
| 15. **Private Subnet** | **PLACE / ISOLATE** — Barrio interno sin ruta directa al IGW; EC2 sin IPv4 pública | VPC > Subnets / Route tables > Subnet associations |
| 16. **Private NACL** | **FILTER SUBNET** — Control del barrio privado; permitir tráfico de aplicación y retorno | VPC > Network ACLs > Inbound / Outbound rules |
| 17. **EC2 Security Group** | **FILTER RESOURCE** — Guardia de oficina: puerto app desde SG del ALB | EC2 > Security Groups > Inbound rule: TCP 80, source ALB SG |
| 18. **ENI / Private IP** | **CONNECT / IDENTIFY** — Conexión y dirección privada de la oficina | EC2 > Instances > Networking > Network interface / Private IPv4 |
| 19. **EC2** | **EXECUTE** — Trabajador que ejecuta Apache, Nginx o aplicación | EC2 > Instances > Launch instances |

## Cinco familias que reconstruyen el TODO

| Familia | Pregunta | Piezas |
|---|---|---|
| **LOCALIZAR** | ¿Dónde está? | Route 53, ALB DNS / IP |
| **CONECTAR** | ¿Cómo llego? | VPC, IGW, subnet, route tables, ENI |
| **PROTEGER** | ¿Me permiten pasar? | NACL (stateless subnet), SG (stateful ENI/recurso) |
| **DISTRIBUIR** | ¿Quién me atiende? | ALB, listener, listener rule, target group |
| **EJECUTAR** | ¿Quién trabaja? | EC2 + aplicación |
| **MANTENER DISPONIBLE** *(transversal)* | ¿Qué pasa si falla o crece? | Multi-AZ, health checks, [[Auto Scaling Group]], [[CloudWatch]], launch template |

## Dos planos: ¡no mezclar!

**Descubrimiento DNS**: Cliente → Route 53 RESOLVE → DNS público ALB.

**Datos HTTPS (modelo conceptual)**: cliente ↔ Internet ↔ IGW / red pública VPC ↔ ALB ENI y su SG ↔ listener / reglas → selección de target healthy ↔ enrutamiento local + NACL / SG ↔ EC2 privada. La VPC y la subnet *contienen* recursos; route tables, NACL y SG no son servidores HTTP.

**Control/observación**: ACM proporciona certificados TLS al listener; target-group health checks comprueban destinos; CloudWatch observa métricas; scaling policies y ASG mantienen/ajustan EC2 registradas en target group; Launch Template define nuevas instancias. Estos componentes no son saltos HTTP.

## Relaciones concretas y decisiones de certificación

- **ALB vs NLB**: ALB L7 para HTTPS, host/path rules; NLB L4 para TCP/UDP y necesidades de IP estática. No instalar ambos por costumbre.
- **NACL vs SG**: NACL stateless en límite de subnet, reglas allow/deny y puertos efímeros de retorno; SG stateful y reglas allow en las interfaces asociadas.
- **IGW vs NAT GW**: IGW conecta VPC con Internet; NAT Gateway opcional permite salida IPv4 iniciada desde private subnet. No se necesita NAT para tráfico entrante ALB → EC2 privada.
- **Target Group vs ASG**: TG registra/comprueba targets y ofrece candidatos al ALB; ASG crea/reemplaza/escala EC2, si está configurado. TG no crea EC2.
- **Multi-AZ**: implementar ALB en al menos dos AZ y distribuir instancias/subnets; no confundir redundancia con cero interrupciones garantizadas.
- **IP del ALB**: usar Alias de Route 53 al DNS del ALB; las IP de nodos de ALB son administradas y pueden cambiar. No fijar una IP pública de EC2 como equivalente.
- **Health check**: el check del target group evalúa endpoint, puerto, código y umbrales; un /health superficial puede no medir dependencias críticas.
- **HTTP 443 → backend 80**: listener HTTPS :443 puede terminar TLS en ALB y enviar HTTP :80 hacia SG de EC2; si se exige cifrado extremo a extremo configurar HTTPS al target.
- **Routing**: la ruta `local` entre subnets de una misma VPC no requiere IGW. Un ALB internet-facing necesita subnets públicas con ruta apropiada al IGW para acceso externo.
- **Configuración no lineal**: primero VPC/subnets/rutas/SG/targets; luego ALB, listener, DNS; esta es una guía de dependencias, no una receta invariable.

## LEGO reconstruible en 10 segundos

**Persona → Guía → Ciudad/Calles → Guardianes → Recepción → Ventanilla → Instrucciones → Directorio saludable → Oficina privada → Trabajador.**

**REQUEST → RESOLVE → CONNECT/ROUTE → FILTER → DISTRIBUTE → LISTEN → DECIDE → GROUP/CHECK → FILTER → CONNECT → EXECUTE.**

## Examen: recuerda la pregunta y descarta el servicio incorrecto

| Si preguntan… | Piensa… | No confundas con… |
|---|---|---|
| ¿Qué nombre apunta al balanceador? | Route 53 Alias | Target Group: no hace DNS |
| ¿Quién decide /api vs /web? | ALB listener rule | Route Table: enruta IP, no paths HTTP |
| ¿Quién comprueba el backend healthy? | Target Group health check | CloudWatch: observabilidad no sustituye health routing |
| ¿Quién reemplaza una EC2 rota? | ASG con health checks configurados | ALB: solo selecciona/rutea |
| ¿Quién autoriza tráfico del ALB a EC2? | SG EC2 con source SG ALB | IAM Role: autoriza API AWS, no acceso TCP |
| ¿Qué expone HTTPS sin hacer pública la EC2? | ALB público + EC2 privada | EC2 con IP pública |

## Relaciones Obsidian / GitHub

Inicio: [[AWS-Onion - LEGO Architecture Map]].
Profundización: [[Elastic Load Balancing - Distribute LEGO Service DNA]], [[ALB - Target Groups and Auto Scaling Functional Onion]], [[EC2 Auto Scaling - Maintain Adjust Functional Onion]], [[AWS-Onion - Verbo Rector y Service DNA]].
El repositorio contiene notas Markdown con wikilinks de Obsidian; para verlas en la aplicación Obsidian es necesario abrir/sincronizar este repositorio como vault o su carpeta de notas.

## Corrección canónica — Simetría de las dos subnets (2026-10-24)

> [!important] La pieza que faltaba
> **Toda subnet tiene una route table asociada**, explícitamente o mediante la main route table de la VPC. Por eso PUBLIC y PRIVATE usan la misma plantilla: **SUBNET → ROUTE TABLE → NACL → SECURITY GROUP**. Es un orden mental de configuración, no una cadena de cuatro dispositivos físicos. El SG se asocia al recurso o ENI, no a la subnet.

| Patrón A–B–C | PUBLIC (recepción ALB) | PRIVATE (EC2 trabajador) |
|---|---|---|
| **A1 SUBNET** | Subnet pública donde se habilita el ALB | Subnet privada donde reside EC2 |
| **A2 ROUTE TABLE** | Ruta local + ruta predeterminada 0.0.0.0/0 hacia IGW | Ruta local; ruta 0.0.0.0/0 a NAT Gateway solo cuando hace falta salida IPv4; **sin ruta directa IGW** |
| **B NACL** | Filtra tráfico en límite subnet pública (stateless) | Filtra tráfico en límite subnet privada (stateless) |
| **C SECURITY GROUP** | Protege ENI/recurso ALB (stateful) | Protege ENI/recurso EC2 (stateful) |

**Evitar doble conteo:** el anterior elemento 14 «VPC Internal Routing» describe la función lógica que la ruta **local** de las tablas de la VPC cumple entre ALB y EC2, mientras que «Private Route Table» es el recurso concreto asociado a la subnet privada. **No son dos route tables que el paquete atraviesa en serie.** Para contar 19 piezas, sustituir el rótulo 14 «VPC Internal Routing» por una nota sobre rutas locales y nombrar **Private Route Table** en el bloque privado. Si se añaden ambos como entradas didácticas, serán 20 rótulos pero no 20 saltos de red.

**Consola:** VPC → Subnets → [Private Subnet] → Route table (verificar asociación, incluso si hereda la principal). También VPC → Route tables → Subnet associations / Routes.

**Regla de memoria:** «Dos barrios, la misma gramática A–B–C; cambian rutas, reglas y destino». La naturaleza pública de la subnet la define la ruta al IGW, no la mera palabra Public del nombre.
