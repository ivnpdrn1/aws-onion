---
type: book-chapter-seed
project: AWS-Onion
updated: 2026-10-23
tags: [book, aws-onion-lego, elb]
---
# Cuando la puerta tiene varios servidores — semilla editorial ELB

## Problema narrativo
Una tienda atendida por un solo empleado: una cola interminable, y si el empleado se enferma, todo se detiene. Agregamos empleados, pero ¿quién asigna cada cliente a alguien disponible? El **ELB** es el coordinador de la entrada, no contrata personal ni hace el trabajo de los empleados.

## LEGO que aparece naturalmente
1. **Route 53 RESOLVE:** encuentra la dirección de la puerta.
2. **Listener ACCEPT:** abre puerta específica (HTTPS/443).
3. **ALB DISTRIBUTE:** interpreta la petición; asigna /orders al equipo de pedidos y /account al de cuentas.
4. **Target Group GROUP:** identifica el equipo de candidatos.
5. **Health Check TEST:** determina quién puede atender con un chequeo definido.
6. **EC2/ECS/Lambda EXECUTE:** realiza la labor.
7. **ASG MAINTAIN/ADJUST:** mantiene número de empleados/instancias según criterios, con health checks ELB habilitados cuando se requiera reposición basada en la app.
8. **CloudWatch OBSERVE:** detecta tendencias y genera métricas/alarmas; no es camino de la solicitud.
9. **Multi-AZ ISOLATE:** distribuye personas entre edificios (dominios de fallo).

## ONION vertical vs LEGO horizontal
```text
VERTICAL: usuario → DNS → listener/ALB → regla → TG → EC2 → app
HORIZONTAL: EC2-A en AZ-A | EC2-B en AZ-B | EC2-C en AZ-C
FUNCIONAL: healthy? → enrutar; unhealthy? → sacar de rotación;
           ASG reponer si está configurado → registrar → verificar → servir
```

## El conflicto didáctico
Cae EC2-A durante una compra. ALB deja de asignarle **nuevas** solicitudes una vez declarada unhealthy. Eso NO garantiza reconectar transparentemente la sesión existente: sin estado externalizado, reintentos e idempotencia, el usuario puede perder su carrito o ver un error. El capítulo debe enseñar el límite de cada LEGO y no prometer FT automática.

## Tres puertas diferentes
- **ALB:** entiende lo que pide el cliente (L7 HTTP).
- **NLB:** mueve conexiones con desempeño y protocolo de red (L4 TCP/UDP, TLS y static IP).
- **GWLB:** entrega flujo a equipos de inspección/firewall mediante GENEVE.

## Decisión arquitectónica
Preguntar primero: ¿web con routing de contenido, transporte TCP/UDP de baja latencia o inspección de seguridad? Elegir ALB/NLB/GWLB por requisito, no porque uno sea universalmente mejor.

## Trama para futuro libro
PROBLEMA → analogía cotidiana → VERBO/OBJETO → RESPONSABILIDAD → relaciones → recorrido normal → falla → recuperación parcial y límites → alternativas descartadas → EXAM CRAM → experimento de laboratorio.

## Laboratorio sugerido
Dos EC2 stateless en dos AZ, User Data instala servidor HTTP con /health; ALB con listener y Target Group; ASG registra targets; probar health checks, retirar una instancia y observar que ELB enruta las nuevas solicitudes y ASG repone según configuración. Registrar evidencia CloudWatch y estudiar límite de sesiones.
