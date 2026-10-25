---
type: book-chapter-draft
updated: 2026-10-24
status: first-draft
tags: [aws-onion, lego, book, architecture]
---
# Una solicitud, cuatro actos: comprender AWS sin memorizar diecinueve cajas

## La promesa al lector
No necesitas recordar de memoria diecinueve servicios. Necesitas reconstruir cuatro preguntas: **¿dónde?, ¿cómo entro?, ¿quién me atiende?, ¿quién trabaja?**. En AWS: **ENCONTRAR → ENTRAR → DISTRIBUIR → EJECUTAR**.

## Acto I. La guía: ENCONTRAR
Una persona escribe el nombre de la empresa. Route 53 funciona como una guía de direcciones: resuelve el dominio al endpoint DNS del Application Load Balancer. La guía no lleva físicamente a la persona; DNS no transporta las solicitudes HTTP. Las IP públicas de los nodos del ALB son administradas por AWS y pueden cambiar. Nunca necesitamos memorizarlas como destino fijo.

## Acto II. La ciudad: ENTRAR
Nuestra VPC es la ciudad delimitada. Internet Gateway permite su conectividad con Internet. Una subnet es un barrio; su route table señala caminos y su NACL filtra cruces del límite del barrio, sin mantener estado. El Security Group es el control de la oficina, asociado a sus interfaces de red y con estado. La recepción del ALB se aloja en subnets públicas con la ruta hacia IGW. Esta arquitectura necesita al menos dos AZ para el ALB público.

## Acto III. La recepción: DISTRIBUIR
El ALB atiende solicitudes. Su listener es la ventanilla que escucha HTTPS 443; el certificado TLS puede venir de ACM. Una listener rule decide: por ejemplo, /api/* se envía al equipo API. Target Group es el directorio de destinos registrados y sus comprobaciones de salud. No es un salto físico adicional. ALB distribuye; no crea instancias ni preserva mágicamente una sesión perdida.

## Acto IV. La oficina: EJECUTAR
La solicitud entra a la red privada usando rutas locales de VPC; no necesita volver a pasar por IGW. El barrio privado repite nuestra gramática: subnet, route table, NACL y SG. En el SG de la EC2 podemos permitir solo tráfico desde el SG del ALB al puerto de la aplicación. La ENI y la dirección IP privada conectan el servidor. EC2 ejecuta Apache, Nginx o cualquier aplicación adecuada.

## El secreto LEGO: una plantilla, dos barrios
**NETWORK ZONE(tipo, destino) = SUBNET + ROUTE TABLE + NACL + SECURITY GROUP.**

En el barrio PUBLIC, el destino funcional es ALB y existe ruta hacia IGW. En el PRIVATE, el destino es EC2 y la comunicación con ALB usa rutas locales. El NAT Gateway, si se necesita, habilita salida IPv4 iniciada desde las instancias privadas; no es requisito del camino de entrada ALB→EC2.

Las cuatro piezas se aprenden una vez y se reutilizan; cambian rutas, permisos y recursos finales. Security Group no es atributo de toda la subnet y la route table no es un servidor físico.

## El quinto problema no es un quinto bloque
¿Y si el trabajador falla? Los health checks ayudan al ALB a elegir targets, CloudWatch observa métricas y Auto Scaling Group, con políticas y salud configuradas, puede crear/reponer EC2 definidas por Launch Templates. Multi-AZ distribuye dominios de fallo. Estos mecanismos son **transversales**, no un paso por el que atraviesa cada solicitud. La aplicación debe diseñar sesiones, estado y reintentos.

## Un mapa para no confundir
- **Descubrimiento:** Route 53 → ALB DNS.
- **Datos:** conexión cliente → ALB internet-facing → target EC2 privada.
- **Control y salud:** health checks, CloudWatch, ASG, Multi-AZ.
- **Seguridad/red:** VPC, rutas, NACL, SG, ENI dan condiciones al flujo; no son una hilera de máquinas independientes.

## Del conocimiento al AWS Console
Route 53 > Hosted zones; VPC > Your VPCs, Internet gateways, Subnets, Route tables, Network ACLs; EC2 > Security Groups, Load Balancers, Listeners and rules, Target Groups, Instances, Auto Scaling Groups; Certificate Manager; CloudWatch. En el libro, cada pantalla debe ilustrar una necesidad ya entendida.

## Reconstrucción y evaluación
1. ¿Por qué Route 53 no es un balanceador?
2. ¿Por qué Public y Private repiten exactamente el tipo de controles pero no las mismas reglas?
3. ¿En qué se diferencian listener rule, target group y Auto Scaling?
4. ¿Por qué EC2 puede permanecer sin IP pública?
5. ¿Qué permanece como punto único de fallo con una sola instancia EC2?

## Continuación editorial
Expandir cada acto con un diagrama arquitectónico, ejercicio real en AWS Console, experimento de fallo y decisión SAA; mantener analogía, precisión y accesibilidad antes que jerga. Unificar la ilustración de la pizarra con estos cuatro bloques. Véase [[AWS-ONION-LEGO - Cuatro Bloques Secuenciales]] y [[AWS-ONION-LEGO - Ciudad Intuitiva y AWS Console]].

## Nota editorial — La ruta que parecía faltar
En el barrio privado también hay **Route Table**. Todo barrio-subnet la tiene, propia o heredada de la principal VPC. Por ello, la plantilla narrativa idéntica es **Subnet + Route Table + NACL + Security Group**. Public usa salida al IGW; Private conserva la ruta local y puede usar NAT para salida IPv4. No representar «VPC Internal Routing» y «Private Route Table» como dos aparatos que se atraviesan: la ruta local pertenece a las tablas. Este descubrimiento es un momento clave del capítulo: dos barrios, una sola gramática.
