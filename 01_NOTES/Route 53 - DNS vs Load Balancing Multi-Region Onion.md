---
type: concept
domain: networking-resilience
updated: 2026-10-16
tags: [aws, aws-onion, route53, dns, alb, multi-region, health-checks, high-availability, disaster-recovery, certification]
---
# Route 53 — DNS vs Load Balancing: Multi-Region AWS Onion

> [!abstract] Tesis
> **Route 53 = WHERE (qué endpoint/región). ELB = WHICH TARGET (a qué backend dentro de la región). ASG = HOW MANY (cuántas instancias). EC2 = EXECUTE.**
>
> DNS **responde consultas de resolución**; **no está en línea con el flujo de datos HTTP**. ELB **recibe y reenvía conexiones/solicitudes**.

## 1. Por qué existen juntos
Al desplegar múltiples instancias y/o regiones para disponibilidad, el cliente debe resolver **dónde** encontrar el servicio y cada región debe decidir **qué target** atenderá la solicitud. Son decisiones complementarias, no servicios rivales.

## 2. Arquitectura multi-región
```text
             Usuario / navegador
                   |
            DNS query: app.com
                   v
          Amazon Route 53 (global)
           /                \
  endpoint Región A    endpoint Región B
        |                    |
   ALB-A regional       ALB-B regional
        |                    |
  Listener / Rules     Listener / Rules
        |                    |
    Target Group         Target Group
       /    \               /    \
     EC2     EC2           EC2     EC2
        ^                    ^
       ASG                  ASG
```
**Importante:** las ramas representan elecciones de endpoint por DNS, no un paquete que cruza Route 53. Los Target Groups son recursos lógicos de configuración/selección, no saltos físicos de paquetes. Los ALB regionales pueden operar en múltiples AZ dentro de su región; **un único ELB no balancea targets de regiones distintas**.

## 3. AWS-Onion: ejes
| Eje | Pregunta | Interpretación |
|---|---|---|
| Vertical | ¿Qué depende de qué? | Cliente → resolución DNS → endpoint ALB regional → listener/reglas → target EC2/app |
| Horizontal | ¿Qué especialidades cooperan? | Resolución/routing global (Route 53); distribución L7 regional (ALB); capacidad (ASG); ejecución (EC2); observabilidad (CloudWatch) |
| Funcional | ¿Qué sucede y en qué orden? | Consulta DNS → respuesta → conexión ALB → evaluación listener/rules → elección de target elegible → app; por separado health checks y escalado |

## 4. Data plane vs control plane
```text
DNS / descubrimiento:
Cliente -> resolvedor DNS -> Route 53 (cuando corresponda)
Cliente <- respuesta DNS (con influencia de cachés y TTL)

Data plane:
Cliente -> ALB (listener) -> EC2 target -> aplicación

Control/observabilidad:
ELB health checks -> estado de los targets
Métricas/CloudWatch + scaling policies -> ASG -> lanza/termina EC2
ASG/ELB -> registra y retira targets en el Target Group
Route 53 health checks / evaluación de alias -> selección de endpoint DNS
```
**No confundir:** el cliente no envía solicitudes HTTP a través de Route 53; ni a través de ASG ni físicamente a través de un Target Group.

## 5. Qué falla y quién responde
| Fallo o necesidad | Diseño relevante | Límite |
|---|---|---|
| Una EC2 no responde | ALB + health checks del Target Group, otros targets sanos | Requiere destinos funcionales |
| Aumenta la demanda | Auto Scaling Group + políticas/métricas | ASG cambia capacidad, no enruta el request |
| Falla una AZ | ALB multi-AZ + EC2 en otras AZ | Debe estar diseñada la redundancia |
| Falla una región | Stack duplicado en otra región + Route 53 failover y chequeos adecuados | DNS no mueve conexiones existentes; TTL/cachés afectan la convergencia |
| Dos regiones atienden a la vez | Route 53 routing apropiado hacia ALB de cada región | Persistencia, base de datos y consistencia deben diseñarse aparte |

## 6. Active-Passive vs Active-Active
**Active-Passive:** Route 53 responde normalmente con endpoint de región primaria saludable y cambia las respuestas futuras al secundario cuando se cumple el criterio de fallo configurado. La región pasiva necesita capacidad y datos según el RTO/RPO buscados.

**Active-Active:** ambas regiones sirven tráfico. Route 53 puede distribuir resoluciones con políticas tales como weighted, latency o geolocation (según requisitos). **Resolución DNS no equivale a distribuir cada petición individual ni garantiza reparto exacto**, debido a caches y TTL.

## 7. Matices que importan para producción y examen
- **Route 53 es DNS global**; puede señalar endpoints AWS y externos. Un registro alias hacia un ALB puede aprovechar evaluación de salud del target, según configuración; no todo failover requiere el mismo tipo de health check.
- **ELB es regional**, mientras que ALB puede distribuir carga entre **múltiples Availability Zones de esa misma región**.
- **ALB** decide a nivel de aplicación HTTP/HTTPS (L7). **NLB** atiende casos de transporte y otras necesidades; no toda pregunta sobre ELB implica ALB.
- Route 53 **no está en línea con el tráfico**; para direccionamiento global por una vía en línea y con direcciones estáticas, evaluar **AWS Global Accelerator** cuando el requisito lo justifique.
- Health check del **Target Group** decide elegibilidad local; health checks/evaluación DNS ayudan a elegir endpoints regionales. **No son el mismo alcance**.
- **DNS failover no es instantáneo**: TTL, caching, conexiones existentes, replicación y disponibilidad de datos limitan la recuperación real.
- **Multi-region exige más que tráfico**: bases de datos, estado de sesión, replicación, IAM, seguridad, despliegues y costes requieren diseño explícito.

## 8. Exam Cram — requisito → arquitectura
| Señal en el escenario | Elijo | Descarto / por qué |
|---|---|---|
| Dirigir usuarios a endpoints en regiones diferentes | Route 53 routing | Un solo ALB multi-region: ELB es regional |
| Repartir HTTP/HTTPS entre instancias en la misma región | ALB + Target Group | Route 53 solo: no balancea cada request entre targets como ALB |
| Añadir instancias por demanda | ASG + scaling policies | ALB solo: distribuye, no aprovisiona cómputo |
| Cambiar de región si primaria no funciona | Route 53 failover + health evaluation + región secundaria preparada | ASG de una sola región: no resuelve fallo regional |
| Enrutar por HTTP path/host | ALB listener rules | DNS solo: no inspecciona paths HTTP |
| Evitar interrupción incluso ante caída regional | Multi-region + estrategia de datos + recuperación probada | Route 53 solo: no replica aplicaciones/datos |

## 9. Procedimiento de razonamiento arquitectónico
1. **Requirement:** ¿Disponibilidad ante fallo de EC2, AZ o región? ¿Latencia? ¿Escala?
2. **Scope:** ¿Global, regional, AZ, VPC, instancia?
3. **Where:** ¿Qué endpoint encuentra el usuario? Route 53.
4. **Which:** ¿Qué backend recibe tráfico? ALB/Target Group.
5. **How many:** ¿Cuánta capacidad? ASG.
6. **Execute:** ¿Qué cómputo sirve la app? EC2 u otro target soportado.
7. **Recovery:** ¿Qué sucede al fallar? Health checks, failover, backups/datos, RTO/RPO.
8. **Evidence:** CloudWatch, métricas, logs, pruebas de recuperación y costos.

## 10. Repaso activo
1. ¿Por qué el navegador no envía todas sus solicitudes HTTP por Route 53?
2. ¿Puede un mismo ALB registrar EC2 de us-east-1 y eu-west-1?
3. ¿Qué aporta un ASG que un ALB no proporciona?
4. ¿Cómo cambia el routing de fallos de instancia vs fallos de región?
5. ¿Qué limita la velocidad real del DNS failover?
6. ¿Por qué no basta Route 53 para conseguir recuperación multi-region?

**Ancla de memoria:** **NAME → PLACE → ENTRY → TARGET → COMPUTE**; es una secuencia conceptual, no una lista literal de dispositivos atravesados. **WHERE → WHICH → HOW MANY → EXECUTE**.

## 11. Relaciones Obsidian
- [[AWS-Onion - Vertical & Horizontal Onion]]
- [[AWS-Onion - Functional Onion]]
- [[ALB - Target Groups and Auto Scaling Functional Onion]]
- [[EC2 - Architecture Patterns and Auto Scaling]]
- [[EC2 - Networking and Connectivity]]
- [[CloudWatch - Mental Model]]
- [[AWS-Onion - Exam Map]]

## 12. Fuente de la clase
Material proporcionado: **DNS vs Load Balancing (1).txt**. Núcleo de la explicación: DNS devuelve direcciones/endpoints y permite dirigir tráfico entre regiones; un ELB recibe el tráfico y lo envía a targets **en la misma región**; Route 53 permite estrategias multi-región Active-Active o Active-Passive basadas en salud.

Las precisiones sobre TTL/cachés, planos de control, Global Accelerator y estado/datos son **ampliaciones arquitectónicas**, no afirmaciones textuales del material.
