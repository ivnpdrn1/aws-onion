# NAT — La salida compartida de la ciudad LEGO

## La pregunta que inicia el capítulo
En nuestra ciudad (VPC) hay oficinas privadas (EC2) que necesitan consultar proveedores externos. No queremos dar a cada oficina una puerta pública propia. ¿Cómo salen, cómo vuelven las respuestas y por qué el exterior ve una única dirección?

## Narración: cinco actos
**Acto 1 — NECESIDAD.** Una oficina privada requiere contactar una API en Internet.
**Acto 2 — ROUTE.** Las señales del barrio (Private Route Table) envían destinos externos al intermediario NAT; las calles internas usan local.
**Acto 3 — TRANSLATE.** El intermediario público mantiene el registro de quién salió y usa su dirección pública Elastic IP para comunicarse.
**Acto 4 — CONNECT.** La Public Subnet permite al NAT llegar al Internet Gateway; la respuesta regresa al intermediario y luego a la oficina que inició el viaje.
**Acto 5 — RESILIENCE.** Si todas las oficinas dependen de la puerta de una sola AZ, pueden perder salida incluso cuando otros barrios siguen en pie. Se distribuyen puertas NAT y rutas por AZ.

## Dos alternativas: conserje gestionado o conserje propio
- NAT Gateway: servicio administrado, escalable, sin SSH ni SG propio.
- NAT Instance: EC2 cuyo dueño debe configurar NAT/forwarding, desactivar source/destination check, actualizar, escalar y recuperar; ofrece personalización.
- Private NAT: no es salida Internet; traduce para comunicar redes privadas mediante infraestructura apropiada.

## La sorpresa pedagógica
Un cliente externo quiere iniciar una visita a la oficina privada. La puerta NAT no acepta esa llegada nueva. Para ofrecer una aplicación web se usa recepción ALB pública, que reenvía hacia la oficina privada por las calles internas. **NAT ≠ ALB.**

## Preguntas de decisión
- ¿Necesito conectividad saliente Internet o entrada de clientes?
- ¿Cuál dirección IP observa el proveedor externo?
- ¿Quién mantiene el intermediario y quién detecta su fallo?
- ¿Dónde están las rutas y qué pasa cuando falla una AZ?
- ¿Se puede evitar NAT con VPC Endpoints cuando el destino es un servicio compatible?

## DNA para el lector
NECESIDAD → PARTITION (Subnet) → ROUTE (Route Table) → TRANSLATE (NAT) → CONNECT (IGW) → RESPOND (API) → RECOVER (Multi-AZ). Explicar límites antes de presentar opciones de examen.

## Fuente
Basado en la transcripción «NAT Gateways and NAT Instances.txt» aportada por el usuario. Analogías, notas sobre alternativas y conexiones editoriales son elaboración propia.

Enlaces: [[Subnet Types - Trafico de ida y vuelta en la ciudad LEGO]], [[NAT Gateway and NAT Instance - Translate LEGO]].
