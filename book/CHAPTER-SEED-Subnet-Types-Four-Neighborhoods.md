# La ciudad de cuatro barrios — Subnets y el viaje de un paquete

## Pregunta que origina la arquitectura
Una empresa quiere una puerta visible desde la calle, empleados que puedan salir a comprar suministros sin exponer sus oficinas, un enlace seguro con la sede y una cámara donde guardar documentos sin rutas exteriores. ¿Cómo diseñar la ciudad (VPC)?

## Analogía LEGO
- VPC: ciudad / rango general.
- Subnet: barrio con direcciones (CIDR); su dirección no decide si tiene salida exterior.
- Route Table: señales de tráfico según **dirección de destino**.
- IGW: salida y llegada directa del exterior para recursos con IP pública habilitada.
- NAT: salida de mensajeros privados y recepción de respuestas; no abre una nueva entrada pública.
- VPN/VGW: conexión privada con otra ciudad.
- Isolated: barrio sin ruta exterior, pero con calles interiores.
- NACL: filtro por barrio (sin memoria); SG: control de acceso por edificio/interfaz (con estado).

## Cuatro escenas — ida y vuelta
**1. Barrio público**: visitante busca la dirección pública, llega por IGW a EC2, esta responde por la ruta Internet. Sin dirección pública, vivir en barrio público no basta.

**2. Barrio privado**: empleado privado inicia viaje por NAT, vuelve su respuesta. Otro visitante no puede entrar espontáneamente por NAT. Para atender visitantes usamos recepción ALB pública que entrega la solicitud a la oficina EC2 privada por vía local.

**3. Barrio VPN**: mensajero de AWS toma la conexión entre ciudades y su contraparte corporativa sabe cómo retornar; sin rutas recíprocas no hay viaje completo.

**4. Barrio aislado**: mensajeros internos sí atraviesan calles locales con permisos; no hay letreros hacia destinos exteriores.

## Prueba editorial
Invitar al lector a escribir cada paquete como «source → destination», localizar la ruta más específica, analizar retorno y luego filtros. Contrastar caminos con y sin IP pública, e introducir por qué NACL sin estado requiere reglas de respuesta.

## Enlace arquitectónico
Capítulo vinculado con el diseño «Una solicitud, cuatro actos», ALB como puerta pública y EC2 privadas como oficinas. **La posición de un recurso en una subnet no determina por sí sola su seguridad**.

## Cuestión de certificación
¿Una EC2 privada sin IP pública puede responder solicitudes HTTP de Internet? Sí, detrás de un ALB público correctamente configurado: el ALB termina la conexión del cliente y crea una conexión privada hacia EC2, sin convertirla en público ni atravesar NAT.

Fuente inicial: diapositiva «Subnet Types» aportada por usuario; escenas y ejemplos son ampliación explicativa AWS-ONION-LEGO.
