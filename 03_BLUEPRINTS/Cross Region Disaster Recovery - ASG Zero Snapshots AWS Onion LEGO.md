---
type: architecture-pattern
project: AWS-Onion
topic: Cross-Region Disaster Recovery with ASG and EBS snapshots
updated: 2026-11-05
tags: [aws-onion-lego, disaster-recovery, auto-scaling, snapshots, multi-region, rto, rpo]
---
# Disaster Recovery entre regiones — ASG en cero y snapshots

## Fuente de la lección vs ampliación arquitectónica
**Lo indicado literalmente por la diapositiva**: requisito «Application on EC2 in an Auto Scaling group requires disaster recovery across Regions». Solución «Create an ASG in a second Region with the capacity set to 0. Take snapshots and copy them across Regions (Lambda or DLM)».

**Ampliación del modelo AWS-ONION-LEGO**: para que una aplicación funcione después de un desastre se necesitan recursos de red, ALB/target groups cuando correspondan, AMI/Launch Template usable en la segunda región, permisos, datos restaurables, validación y mecanismo de corte de tráfico. La diapositiva no define tiempos RTO/RPO ni asegura failover automático.

## Problema y decisión
Multi-AZ protege ante fallos de una zona de disponibilidad, **no** de toda una región. ASG es un recurso regional. Una región secundaria con ASG Desired Capacity = 0 mantiene una configuración preparada pero **ninguna instancia EC2 de ese grupo en ejecución**, reduciendo costos de cómputo en espera y aceptando mayor tiempo de recuperación.

## VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN
| LEGO | VERBO / OBJETO | RESPONSABILIDAD Y LÍMITE |
|---|---|---|
| Región secundaria | ISOLATE / fallo regional | Dominio independiente, no se activa automáticamente por existir |
| Snapshot EBS | PRESERVE / bloques de volumen | Conserva copia puntual; no ejecuta aplicaciones ni necesariamente restaura coherencia transaccional |
| DLM o Lambda | AUTOMATE / backups | Programa snapshots/copia entre regiones; ver permisos, cifrado/KMS y retención |
| Launch Template / AMI | DEFINE / arranque EC2 | Debe existir imagen y configuración **válidas en región B**; copiar snapshot no crea por sí solo AMI o Launch Template listo |
| ASG B (Desired=0) | MAINTAIN / capacidad | Configurado sin EC2 mientras espera; para recuperar ajustar capacidad, típicamente con Min/Desired/Max adecuados |
| EC2 B | EXECUTE / aplicación | Las nuevas instancias deben disponer de código, dependencias, red y datos |
| ALB + Target Group B | DISTRIBUTE + CHECK / requests/targets | Enruta únicamente cuando el backend está listo; requiere preparación regional |
| Route 53 | RESOLVE / DNS | Puede usar estrategia failover, pero requiere configuración de health/failover y endpoint saludable; no restaura EC2 |
| CloudWatch / alarmas | OBSERVE / estado | Da visibilidad; activación automática exige orquestación explícita |
| RTO / RPO | MEASURE / tiempo y pérdida de datos | Condicionan frecuencia de copias, automatización, costo y arquitectura |

## Diagrama ASCII (control y datos separados)
```text
                  USUARIO / www.app.com
                           |
                       ROUTE 53
                           |
                  ALB REGIÓN A (activa)
                           |
                   TG + ASG A + EC2
                           |
                SNAPSHOT EBS / BACKUP
                           |
                    COPY CROSS-REGION
                           v
                 REGIÓN B (recuperación)
                 respaldos disponibles
                 AMI / Launch Template
                 VPC / subnets / SG
                 ALB / TG preparados (*)
                 ASG B Desired = 0
                           |
                   SI FALLA REGIÓN A
                           v
                 DETECT / DECLARE desastre
                           |
                 RESTORE datos necesarios
                           |
                 SCALE ASG B (0 -> N)
                           |
                 BOOTSTRAP / APP READY
                           |
                 TG HEALTH CHECK = OK
                           |
                 ROUTE 53 FAILOVER a B
                           |
                   SERVICIO RESTAURADO

(*) Recursos pueden preexistir o crearse al recuperar:
    cambia el RTO, complejidad y costo.

CONTROL PLANE: backups, alarmas, scale, orquestación.
DATA PLANE: cliente -> DNS -> ALB B -> target EC2 B.
```

## Secuencia funcional de recuperación
1. Definir de antemano RTO y RPO, datos críticos, pruebas y quién declara desastre.
2. Crear snapshots/coherencia requerida y verificar su copia y accesibilidad en B (KMS cross-region si aplica).
3. Preparar/rediseñar AMI, Launch Template, subnets, SG, roles, ALB/TG y dependencias de la región B; no asumir que todo cruza regiones.
4. Ante desastre, restaurar datos/configuración de aplicaciones (algunos sistemas necesitan bases de datos o replicación específicos, no solo EBS).
5. Activar ASG secundario mediante Desired Capacity > 0 con límites compatibles y lanzar EC2.
6. Comprobar bootstrap y health checks; luego dirigir usuarios al servicio B mediante mecanismo configurado de failover (Route 53 u otro) y validar funciones de negocio.
7. Medir RTO/RPO reales y planificar failback, evitar doble escritura o split-brain.

## Alternativas según RTO/RPO y costo
- **Backup & Restore**: conservar respaldos y aprovisionar infraestructura al recuperar; menor costo, normalmente más RTO.
- **Pilot Light**: mantener componentes esenciales mínimos activos (a menudo datos/servicios críticos), escalar capa de aplicación durante el evento. **La diapositiva por sí sola no demuestra que esos componentes esenciales estén corriendo**, por lo que no clasificar automáticamente como Pilot Light.
- **Warm Standby**: entorno secundario activo, reducido; más costo, posible RTO menor.
- **Active-Active Multi-Region**: regiones atienden usuarios simultáneamente; mayor complejidad/costo y exigencia de consistencia.

**Precisión**: snapshot cada seis horas no garantiza RPO de exactamente seis horas: considerar consistencia, última copia exitosa, replicación y validación.

## Certificación — preguntas de decisión
- ¿Qué resuelve Multi-AZ vs Multi-Region?
- Si ASG B = 0, ¿qué recursos consumen costos y qué limita el RTO?
- ¿Por qué copiar un snapshot no equivale a recuperar una aplicación?
- ¿Cómo garantizar que la ruta Route 53 a B no apunte antes de que TG esté healthy?
- ¿Cuándo elegir backup/restore, pilot light, warm standby o active-active?

## Analogía para el libro
La tienda A opera con empleados. La tienda B tiene planos, suministros y fotografías actualizadas del inventario, pero cero empleados contratados. Ante un incendio, se contratan empleados, se prepara la tienda, se restaura el inventario y finalmente se indica a los clientes una nueva dirección. **Tener el plano y copias no equivale a abrir la tienda inmediatamente**.

Enlaces: [[EC2 Auto Scaling - Maintain Adjust Functional Onion]], [[Elastic Load Balancing - Distribute LEGO Service DNA]], [[High Availability Fault Tolerance Durability - AWS Onion LEGO]].
