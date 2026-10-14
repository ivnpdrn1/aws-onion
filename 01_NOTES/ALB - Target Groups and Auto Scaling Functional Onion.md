---
type: concept
domain: compute-networking
updated: 2026-10-14
tags: [aws, alb, target-group, auto-scaling, cloudwatch, eni, aws-onion, exam]
---
# ALB — Target Groups and Auto Scaling Functional Onion

> **ALB = a cuál destino; Target Group = cuáles destinos están configurados y saludables; ASG = cuántas instancias existen; CloudWatch = qué ocurre.**

## ¿Por qué existe Target Group?
El Application Load Balancer recibe solicitudes en un **listener** (ej. HTTPS:443). Las reglas del listener pueden **forward** hacia un **Target Group**. Este es un **recurso lógico de Elastic Load Balancing**, no una EC2, subnet, ENI ni un salto de red físico; mantiene registro y estado de salud de destinos y la configuración para alcanzarlos. Permite sustituir instancias sin reprogramar las reglas ALB cada vez.

## Dos secuencias distintas: Data Plane y Control Plane

```text
SOLICITUD REAL:
Cliente HTTPS:443 → ALB (listener y reglas)
                    → decisión usando Target Group HTTP:80
                    → EC2 saludable (ENI + IP privada + app HTTP:80)

ADMINISTRACIÓN / ESCALADO:
CloudWatch métricas → política dinámica (o evento programado / acción manual)
                    → Auto Scaling Group (min/desired/max)
                    → lanza o retira EC2 desde Launch Template
                    → ASG registra/desregistra instancia del Target Group
                    → ELB comprueba salud del target
                    → ALB puede enviar solicitudes a nuevo destino saludable
```

**No se envían paquetes de usuario a través del ASG ni de un servidor físico denominado Target Group.** El ASG opera en el plano de control; el ALB encamina solicitudes hacia los targets.

## Tabla: necesidad → recurso → implementación → eslabón siguiente

| Eslabón | ¿Por qué? | ¿Cómo se crea/configura? | Se enlaza con |
|---|---|---|---|
| 1. Cliente | Origina petición | DNS/URL HTTPS | ALB DNS/listener |
| 2. ALB | Punto de entrada y balanceo | Crear Application Load Balancer en VPC/subnets de al menos dos AZ; configurar Security Group | Listener |
| 3. Listener + Rule | Decide acción por puerto, host o path | Listener HTTPS:443 y regla forward | Target Group |
| 4. Target Group | Lista lógica de targets elegibles; health checks | Crear target type=instance, protocol HTTP:80, health path `/health`, VPC correcta | EC2 registradas |
| 5. EC2 + ENI | Ejecuta la aplicación | Launch Template: AMI, tipo, IAM role, subnets, SG, user data cuando aplica | App en puerto 80 |
| 6. ASG | Ajusta número de EC2 y mantiene registro | Crear ASG (min/desired/max); asociar Target Group ARN | Nuevos EC2 targets |
| 7. CloudWatch/política | Señales para scaling dinámico | Target tracking (ej. CPU o ALBRequestCountPerTarget) o step alarms | ASG desired capacity |

### Controles de red
- SG del ALB permite entrada HTTPS:443 de los clientes apropiados y salida al puerto de los targets.
- SG de EC2 permite entrada HTTP:80 **desde SG del ALB**; no hace falta hacer pública la EC2.
- NACL/rutas deben permitir tráfico entre ALB y EC2, incluidas respuestas y health checks.
- ALB puede ser internet-facing en subnets públicas y EC2 estar en privadas.

## ¿Y si el Target Group no existe?
- Un listener de ALB puede devolver una respuesta fija o redirigir, pero para **forward hacia EC2** necesita un Target Group configurado.
- El ASG puede funcionar sin ALB ni Target Group (ejemplo: workers SQS).
- Un ALB puede servir EC2 fijas sin ASG.
- Si el target group existe pero **no tiene targets elegibles**, no habrá destinos normales donde reenviar el tráfico; ELB puede responder error 503. Hay comportamiento **fail-open** cuando todos los targets registrados están unhealthy: no confundir con tener un Target Group vacío.

## ¿Qué sucede al aumentar la demanda?
| Orden lógico | Responsable | Efecto |
|---|---|---|
| 1 | Clientes / ALB | Solicitudes llegan a listener y se reparten |
| 2 | CloudWatch | Registra métricas (CPU, requests/target, etc.) |
| 3 | Política de scaling | Evalúa condición para escalar |
| 4 | ASG | Ajusta desired capacity dentro de min y max |
| 5 | EC2/ASG | Lanza instancias y las registra en Target Group asociado |
| 6 | Elastic Load Balancing | Aplica Health Checks; identifica targets saludables |
| 7 | ALB | Incluye destinos saludables en decisiones de distribución |

Las comprobaciones de salud ELB pueden habilitarse adicionalmente como fuente para reemplazo de instancias por ASG. El health check path debe responder de manera fiable y representa la preparación real de la app.

## Reglas mentales / Exam Cram
- **Which target? ALB**; **eligible + healthy? Target Group**; **how many EC2? ASG**; **metric observations? CloudWatch**.
- **Listener** acepta tráfico; **Rule** decide forward; **Target Group** registra targets; **Health Check** evalúa; **ENI** recibe en EC2; **ASG** crea/retira capacidad.
- **Target Group no es salto físico.**
- **ALB no tiene que ir «antes» del ASG en orden de creación:** se relacionan pero no constituyen etapas seriales del tráfico.
- **Target Tracking** administra alarmas de CloudWatch; **Scheduled/Manual Scaling** no necesita alarmas de demanda.
- **Availability** se consigue con ALB multi-AZ, instancias saludables, ASG y políticas adecuadas; no con Target Group solo.

## Relaciones Obsidian
- [[EC2 - Architecture Patterns and Auto Scaling]]
- [[EC2 - Networking and Connectivity]]
- [[CloudWatch - Mental Model]]
- [[AWS-Onion - Functional Onion]]
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[AWS-Onion - Mental Model]]

## Lectura AWS
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html
- https://docs.aws.amazon.com/autoscaling/ec2/userguide/attach-load-balancer-asg.html
- https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html
