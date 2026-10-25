---
type: core-teaching-model
updated: 2026-10-24
tags: [aws-onion, lego, architecture, study, book]
---
# AWS-ONION-LEGO — Cuatro bloques secuenciales

> [!important] Formula madre
> **ENCONTRAR → ENTRAR → DISTRIBUIR → EJECUTAR.**
> Los 19 componentes anteriores permanecen en [[AWS-ONION-LEGO - Ciudad Intuitiva y AWS Console]] como catálogo detallado. El objetivo pedagógico es memorizar cuatro responsabilidades y reconstruir el resto, no memorizar diecinueve cajas.

## Mapa de bloques

| Bloque | Pregunta humana | Verbo | Piezas | AWS Console |
|---|---|---|---|---|
| 1. ENCONTRAR | ¿Dónde está? | RESOLVE | Cliente → Route 53 → Alias al DNS de ALB | Route 53 > Hosted zones > Records; EC2 > Load Balancers > DNS name |
| 2. ENTRAR | ¿Cómo llego y qué controles autorizan? | CONNECT, ROUTE, FILTER | VPC, IGW y NETWORK ZONE(PUBLIC, ALB) | VPC > Your VPCs / Internet gateways / Subnets / Route tables / Network ACLs; EC2 > Security Groups |
| 3. DISTRIBUIR | ¿Quién atiende? | LISTEN, DECIDE, GROUP/CHECK, DISTRIBUTE | ALB, HTTPS Listener, Listener Rule, Target Group | EC2 > Load Balancers > Listeners and rules; EC2 > Target Groups; ACM |
| 4. EJECUTAR | ¿Cómo llego a la oficina y quién trabaja? | ROUTE, FILTER, CONNECT, EXECUTE | NETWORK ZONE(PRIVATE, EC2), ENI/Private IP, EC2/App | VPC > Subnets / Route tables / Network ACLs; EC2 > Security Groups / Instances > Networking |

## Patrón reutilizable: NETWORK ZONE(tipo, destino)

**SUBNET → ROUTE TABLE → NACL → SECURITY GROUP** es un orden didáctico de comprobación, no el tránsito literal de un paquete. La tabla de rutas se asocia a la subnet; NACL regula su frontera de forma stateless; SG protege ENI/recurso de forma stateful.

| Atributo común | PUBLIC (ALB) | PRIVATE (EC2) |
|---|---|---|
| Subnet | Subnets públicas en al menos dos AZ para el ALB | Subnets privadas para las EC2, preferiblemente Multi-AZ |
| Route table | Rutas locales + ruta 0.0.0.0/0 hacia IGW | Ruta local para acceso VPC; NAT GW opcional para salida IPv4 |
| NACL | Filtra frontera pública, considerar tráfico de retorno | Filtra frontera privada, considerar tráfico de retorno |
| Security Group | Permite HTTPS 443 de clientes | Permite puerto backend desde el SG del ALB |
| ENI/IP | ALB usa interfaces privadas y endpoint DNS con IP públicas administradas | EC2 usa ENI/IP privada sin IP pública |
| Destino | Recepción/distribución | Oficina/ejecución |

**Lo común:** (A) subnet + tabla asociada, (B) NACL, (C) SG. **Lo variable:** rutas, permisos, zona, dirección y recurso final.

## La historia en cuatro actos
1. El cliente conoce el nombre. Route 53 resuelve hacia el ALB, no transporta HTTP.
2. La ciudad VPC, la entrada IGW, las subnets, rutas y guardianes NACL/SG permiten alcanzar la recepción.
3. El ALB escucha por su listener (HTTPS:443), interpreta reglas como /api/* y usa el Target Group para seleccionar destinos elegibles, según health checks.
4. La ruta interna local y los permisos del barrio privado permiten llegar a la ENI/IP de EC2, donde se ejecuta la aplicación (Apache, Nginx u otra).

## Plano DNS, datos y control: no confundir
- **DNS:** cliente → Route 53 → endpoint ALB.
- **Tráfico:** cliente → ALB público por conectividad VPC/IGW; ALB → EC2 privada por red interna y controles aplicables. El IGW NO está entre ALB y EC2.
- **Control:** health check del Target Group; CloudWatch OBSERVE; scaling policy y Auto Scaling Group MAINTAIN/ADJUST con Launch Template; Multi-AZ ISOLATE FAILURES. Nada de esto representa saltos HTTP.
- **Certificado:** ACM y listener HTTPS; no requiere inventar un recurso por capa.

## Decisiones y alternativas descartadas
- HTTP host/path routing: **ALB**, no NLB para path rules.
- EC2 privada sirviendo Internet: **ALB público + SG del backend con origen SG de ALB**, no IP pública obligatoria en EC2.
- Escalar/reponer: **ASG**, no TG; TG agrupa y comprueba salud, pero no crea EC2.
- Aislar subnet vs recurso: **NACL stateless** frente a **SG stateful**; IAM Role autoriza API AWS, no conexiones TCP.
- Acceso a Internet desde EC2 privada: **NAT GW opcional**; no aparece en el flujo de entrada ALB→EC2.
- Tolerancia ante caída de AZ: repartir ALB y capacidad EC2 entre AZ; ALB solo no evita SPOF si hay una EC2.
- Un ALB no garantiza continuidad de sesiones si el backend falla; pensar estado, reintentos y persistencia.

## Orden razonable de construcción en consola
1. VPC y subnets públicas/privadas Multi-AZ; IGW, route tables, NACL.
2. SG de EC2, EC2/Launch Template y aplicación con ruta de health.
3. Target Group y ALB con SG, listener HTTPS y reglas.
4. Certificado ACM (validado antes de vincular listener), Route 53 Alias, ASG, CloudWatch y ensayos de fallo.

No confundir orden de aprovisionamiento con el mapa conceptual.

## Regla de libro y enseñanza
Cada concepto: **NECESIDAD → ANALOGÍA → VERBO/OBJETO → RESPONSABILIDAD → RELACIONES → AWS CONSOLE → ALTERNATIVAS → FALLO → EXAMEN**.

### Vinculaciones
[[AWS-Onion - LEGO Architecture Map]] · [[AWS-ONION-LEGO - Ciudad Intuitiva y AWS Console]] · [[Elastic Load Balancing - Distribute LEGO Service DNA]] · [[EC2 Auto Scaling - Maintain Adjust Functional Onion]] · [[AWS-ONION-LEGO - Capitulo Cuatro Actos]]
