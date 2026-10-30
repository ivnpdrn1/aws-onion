# AWS-ONION-LEGO — ALB en cinco bloques

> **Modelo didáctico:** el mismo ALB aparece como endpoint DNS (03), recurso protegido por su Security Group (09) y distribuidor de solicitudes (10). No son tres balanceadores. El diagrama representa asociaciones y responsabilidades, no una secuencia física literal de todos los filtros y rutas.

```text
BLOQUE 1 — DISCOVER
[01] CLIENTE / INTERNET
  |
[02] ROUTE 53 — RESOLVE www.miapp.com
  |
[03] ALB DNS / PUBLIC ENDPOINT (alias, IPs gestionadas por AWS)
  |
  v
BLOQUE 2 — CONNECT (VPC, red y controles)
[04] INTERNET GATEWAY (IGW)
  |
[05] VPC
  |
  +--------------------------+--------------------------+
  | AZ-A                     | AZ-B                     |
[06] PUBLIC SUBNET A       [06] PUBLIC SUBNET B
[07] PUBLIC ROUTE TABLE A  [07] PUBLIC ROUTE TABLE B
[08] PUBLIC NACL A         [08] PUBLIC NACL B
  |                          |
  +------------+-------------+
               |
[09] ALB SECURITY GROUP — permitir HTTPS 443 de clientes
               |
               v
BLOQUE 3 — DISTRIBUTE
[10] APPLICATION LOAD BALANCER
  |
[11] LISTENER HTTPS :443
  |
[12] LISTENER RULE (host/path, ej. /orders)
  |
[13] TARGET GROUP (targets + health check)
  |
  +--------------------------+--------------------------+
  | AZ-A                     | AZ-B                     |
  v                          v
BLOQUE 4 — EXECUTE
[14] INTERNAL VPC CONNECTIVITY / local routing
[15] PRIVATE SUBNET A      [15] PRIVATE SUBNET B
[16] PRIVATE ROUTE TABLE A [16] PRIVATE ROUTE TABLE B
[17] PRIVATE NACL A        [17] PRIVATE NACL B
[18] EC2 SECURITY GROUP    [18] EC2 SECURITY GROUP
[19] PRIVATE IP / ENI      [19] PRIVATE IP / ENI
[20] EC2-A / APP           [20] EC2-B / APP

BLOQUE 5 — CONTROL / OBSERVE / RECOVER
Target Group Health Checks -> ELB selecciona destinos saludables
CloudWatch -> métricas/alarmas -> Scaling policy
ASG -> MAINTAIN/ADJUST EC2 según configuración
Multi-AZ -> distribuye dominios de fallo
(No son saltos adicionales del request HTTP)
```

## Notas técnicas fundamentales
- El ALB vive en **al menos dos subnets de diferentes AZ** para el diseño típico de ALB público; los targets EC2 pueden alojarse en subnets privadas distribuidas.
- Route 53 resuelve el DNS; no inspecciona el path HTTP ni reenvía directamente cada request.
- Listener y listener rules pertenecen a la configuración del ALB; Target Group mantiene el conjunto de destinos y define health checks.
- El SG del ALB permite la entrada HTTPS; el SG de EC2 debe permitir el puerto de aplicación/health check desde el SG del ALB según el diseño.
- Cada subnet tiene una asociación con una Route Table; para el tráfico interno de la VPC normalmente rige la ruta `local`.
- La ruta hacia IGW hace pública una subnet, junto con requisitos de direccionamiento y conectividad; el ALB público administra sus IP públicas, no representa una IP fija única.
- NACL (stateless) filtra a nivel subnet; Security Group (stateful) protege las interfaces/recurso. Su representación vertical es conceptual, no una descripción de cajas físicas en cadena.
- En caso de fallo: TG health check detecta según umbrales, ELB deja de seleccionar targets unhealthy si hay targets sanos, ASG puede reemplazar si está configurado para usar ELB health checks. CloudWatch no es necesariamente el intermediario del reemplazo.
- ALB + Auto Scaling + Multi-AZ no garantizan por sí solos preservar sesiones, RAM o transacciones; externalizar estado y diseñar recuperación/reintentos idempotentes.

## Recordatorio para examen y futuro libro
**FIND → REACH → ALLOW → DISTRIBUTE → EXECUTE**, acompañado por **CHECK → OBSERVE → MAINTAIN**.
La analogía: una misma puerta tiene dirección pública, control de acceso y recepcionista; el trabajo ocurre detrás de ella. Cada una es una perspectiva distinta sobre el mismo ALB.

Relacionado: [[Elastic Load Balancing - Distribute LEGO Service DNA]], [[ELB - Six Round Learning Gaps and Reinforcement AWS Onion LEGO]].
