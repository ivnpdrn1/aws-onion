# ELB — Refuerzo posterior a seis rondas AWS-ONION-LEGO

Resultados: 6/10 → 6/10 → 7/10 → 9/10 → 9/10 → 10/10. Nivel 1 de selección múltiple superado. Nivel 2 de diseño libre pendiente.

## P1: Inspección de red frente a routing web (errores recurrentes)
- GWLB distribuye tráfico hacia appliances especializadas; ALB enruta solicitudes HTTP por host/path.
- Diseñar ambos cuando la arquitectura requiere inspección de red y microservicios HTTP; no atribuir routing por /api a GWLB.
- GWLB Endpoints y rutas deben configurarse explícitamente.

## P2: DNS no es conectividad
- Route 53 resuelve nombre; ALB recibe y enruta; TG comprueba salud; SG/NACL/rutas permiten el camino; la app responde.
- DNS exitoso no demuestra backend disponible. ASG no arregla automáticamente un SG defectuoso.

## P3: Elección por protocolo
- ALB: HTTP/HTTPS L7. NLB: TCP/UDP y contratos de IP fija/baja latencia. GWLB: inspección por appliances.

## P4: Sesión y transacciones
- ELB distribuye; ASG repone EC2; ninguno garantiza conservar RAM, sesión o transacción.
- Externalizar estado y aplicar idempotencia en operaciones con reintentos, especialmente pagos.

## P5: Alta disponibilidad del TODO
- EC2 Multi-AZ no elimina SPOF de DB, Redis, autenticación o almacenamiento.
- Probar fallos controlados de AZ y comprobar RTO, RPO y operaciones esenciales.

## Método para secciones futuras
1. VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN.
2. Rondas de 10 preguntas hasta 10/10; catalogar errores por causa.
3. Cinco escenarios abiertos sin alternativas más un laboratorio o arquitectura dibujada.
4. Revisar qué alternativas se descartan y por qué.

## Material para el libro
Analogías: la dirección correcta pero la puerta cerrada (DNS), el portero frente al inspector de seguridad (ALB/GWLB), sustituir empleados sin recuperar el cuaderno (ASG/estado), dos tiendas con una sola caja fuerte (Multi-AZ y SPOF).
