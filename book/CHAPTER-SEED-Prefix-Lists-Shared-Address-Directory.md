# Prefix Lists — El directorio compartido de la ciudad LEGO

## Pregunta inicial
Cada oficina de una ciudad mantiene numerosas copias de un directorio con las mismas direcciones. Cuando cambia una calle, los porteros y las señales deben corregir decenas de documentos diferentes. ¿Y si todos consultaran una lista central compartida?

## Necesidad → aparición de la pieza
La **Prefix List** no es una puerta, un portero ni una carretera; es **un directorio de barrios (rangos CIDR)** al que hacen referencia otras piezas. En vez de mantener las mismas direcciones múltiples veces, se mantiene un conjunto reutilizable.

## Personajes — funciones, verbos, límites
- CIDR DESCRIBE el barrio/intervalo de direcciones.
- Prefix List GROUP / REFERENCE muchos CIDR.
- Security Group ALLOW: portero que acepta visitantes según fuente y puerto. La lista alimenta la regla.
- Route Table ROUTE: señal de carretera que decide Target según destino. La lista agrupa destinos.
- AWS-managed MAINTAIN BY AWS: directorio oficial de rangos de servicios como CloudFront, S3 y DynamoDB.
- Customer-managed MAINTAIN BY CLIENT: directorio corporativo propio.
- NAT TRANSLATE y Gateway VPC Endpoint CONNECT: no confundir funciones de transporte con una lista informativa.

## Escena 1: CloudFront y el portero del ALB
El ALB acepta HTTPS solo desde rangos de CloudFront usando la lista administrada como Source. **Matiz:** un directorio de calles CloudFront no es credencial de identidad de nuestra distribución; conviene controlar también la autenticidad de la llegada al origen.

## Escena 2: mapa de carreteras
Una Route Table usa la lista que agrupa rangos de tres sedes para enviar tráfico al mismo Transit Gateway. La lista representa Destination, **no Target**. Las rutas de retorno y las reglas de acceso siguen siendo necesarias.

## Escena 3: S3 y DynamoDB sin viaje al NAT
El mapa de direcciones del servicio (AWS-managed Prefix List) puede vincularse a una ruta de Gateway VPC Endpoint. La lista identifica las direcciones; el endpoint realiza la conexión. Si la oficina consulta una API pública de otra empresa, necesita un camino externo apropiado: no basta una Prefix List.

## Escena 4: cambios y costo invisible
Centralizar IP ahorra modificaciones, pero un cambio del directorio afecta simultáneamente múltiples consumidores. Además, en cuotas de Security Group una lista puede tener peso/capacidad: una referencia visual no siempre consume «1».

## Secuencia pedagógica del capítulo
NECESIDAD → GROUP → REFERENCE → ALLOW o ROUTE → DECISIÓN → LÍMITES → EXAMEN.
Distinción capital: un **descriptor reutilizable** (Prefix List) y un **componente que ejecuta** acciones (SG / RT / NAT / Endpoint) no son equivalentes.

## Fuentes y evidencia pedagógica
Material original «Prefix Lists.txt»: concepto, simplificación, AWS-managed CloudFront/S3/DynamoDB, customer-managed, SG y RT, IPv4/IPv6. Capítulos sobre CloudFront origin hardening, Gateway Endpoints, cuotas y criterios de rutas son **ampliaciones arquitectónicas**.
Evaluaciones verificadas: Ronda 1 10/10 y Ronda 2 10/10.
Conectar con [[Subnet Types - Trafico de ida y vuelta en la ciudad LEGO]], [[NAT - La salida compartida de la ciudad LEGO]] y [[Prefix Lists - Group Reference Reuse LEGO]].
