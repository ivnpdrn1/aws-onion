# Semilla de capítulo — Cuando una pieza LEGO falla

**Proyecto:** AWS-ONION-LEGO: aprender AWS como arquitectura natural, intuitiva y reconstruible.

## Tesis narrativa
Una aplicación con una única EC2 puede funcionar de maravilla, hasta el primer fallo. El arquitecto no busca una EC2 inmortal: pregunta **qué sucede con el TODO si retiro esta pieza**.

## Secuencia didáctica del capítulo
1. **Escena:** usuario llega a una aplicación; una sola EC2 es un punto único de fallo.
2. **Necesidad:** interrupción mínima. Agregamos redundancia Multi-AZ, no por moda sino por dominio de fallo.
3. **Reparto de funciones:** Route 53 RESOLVE, ALB DISTRIBUTE, Target Group GROUP/CHECK, EC2 EXECUTE, ASG MAINTAIN/ADJUST, CloudWatch OBSERVE.
4. **Giro:** falla una EC2; ALB la deja de usar tras health checks, ASG la reemplaza. Algunos usuarios pueden percibir interrupción.
5. **Segundo giro:** cae una AZ; las instancias de la otra AZ siguen si hay suficiente capacidad y las demás dependencias resisten.
6. **Elección:** HA minimiza interrupción; FT busca continuidad sin interrupción para fallos definidos; durability preserva datos; availability permite acceder.
7. **Transferencia:** sustituye Apache por Nginx; el verbo SERVE sigue igual, las piezas de resiliencia permanecen.
8. **Examen y vida real:** distinguir necesidad, límites, coste y qué componente NO realiza cada función.

## Arquitectura que el lector reconstruye
```text
DNS query: Route 53 -> ALB endpoint
Client traffic: Client -> ALB -> TG -> EC2 in AZ-A + AZ-B
Control: Launch Template -> ASG -> maintain healthy EC2 capacity
Signals: health checks, metrics/alarms -> control decisions
State: database/session/storage must be independently resilient
```

## Promesa editorial
No presentar listas de servicios desconectados. Para cada pieza: **problema humano → VERBO → OBJETO → RESPONSABILIDAD → relaciones aguas arriba/abajo → qué NO hace → fallo → alternativa → criterio de decisión → experimento mental → pregunta de examen**.

## Precisión que debe preservarse
- Un diagrama funcional no es ruta física literal.
- Los objetivos FT cero interrupción/cero pérdida se limitan a escenarios y garantías explícitas.
- Route 53 resuelve DNS; no atraviesa todos los paquetes.
- AWS administra redundancia subyacente; la aplicación sigue siendo responsabilidad arquitectónica del cliente.
- Los ejemplos docentes no reemplazan garantías contractuales de servicios AWS.

Material detallado: [nota LEGO](../01_NOTES/High%20Availability%20Fault%20Tolerance%20Durability%20-%20AWS%20Onion%20LEGO.md).
