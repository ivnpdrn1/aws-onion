# VPC Gateway Endpoint — La carretera especial hacia S3

## Problema que justifica una nueva pieza
Una ciudad LEGO AWS tiene oficinas EC2 en barrios privados; una de ellas necesita documentos almacenados en Amazon S3. Ya aprendimos a salir por NAT→Internet Gateway, pero ¿por qué recorrer la salida general para visitar un servicio AWS con conectividad específica disponible?

## Analogía: cuatro elementos, cuatro responsabilidades
1. **EC2 REQUEST:** oficina que quiere obtener el documento.
2. **DNS RESOLVE:** busca la dirección del archivo/servicio.
3. **PREFIX LIST GROUP/IDENTIFY:** el directorio de direcciones de S3; no es carretera.
4. **ROUTE TABLE ROUTE:** el mapa de carreteras, lee la dirección de destino, compara los rangos y elige la vía.
5. **GATEWAY ENDPOINT CONNECT:** la conexión especial creada para llegar a S3; no es el mapa ni el destino final.
6. **POLICIES AUTHORIZE:** las credenciales y los permisos de la oficina, del corredor y del archivo.
7. **S3 EXECUTE/RESPOND:** archivo corporativo que entrega o deniega el documento.

## Escena clave del capítulo: Destination vs Target
Los términos parecen sinónimos en el idioma ordinario, pero en la Route Table:
- **Destination = WHERE = S3 IPs representadas por `pl-...`**.
- **Target = VIA = S3 Gateway Endpoint `vpce-...`**.

```text
ROUTE TABLE
Destination              Target
pl-63a5400a       →       vpce-0302c9774934f0bfa
(WHERE: S3 CIDRs)         (VIA: gateway endpoint)
10.0.0.0/16       →       local
```
Cuando en la consola se hace clic en `pl-`, se abre el directorio de CIDR (Managed Prefix Lists); cuando se hace clic en `vpce-`, aparece el recurso de conexión (Endpoints). **Eso confirma ambas columnas**. No interpretar un hipervínculo como flujo físico de datos.

## Demostración experimental narrativa
Primero una EC2 pública consulta S3 por IGW: `aws s3 ls` funciona si IAM permite. El director restringe el nuevo corredor (Endpoint Policy Deny) pero la EC2 todavía puede consultar por IGW, porque su mapa no usa el corredor. Cuando asociamos su Route Table al Gateway Endpoint, el mapa incorpora la ruta `pl-S3→vpce-`; el acceso falla por Deny. Finalmente se restituye Allow y la conexión vuelve a funcionar por el camino específico.

## Moraleja del arquitecto
**No basta crear una infraestructura: hay que seleccionar el camino, demostrar que se usa y distinguir conectividad de autorización.**
Una Route Table no fabrica la conexión; un Endpoint no decide para qué IPs la subnet debe elegirlo sin la ruta asociada. La Prefix List no transporta paquetes y las políticas no son servidores intermedios.

## Sección futura del libro
- Historial pedagógico: Subnets → NAT → Prefix Lists → S3 Gateway Endpoint.
- Matriz «IP destino → Route Table → Target → conectividad → IAM/Endpoint/Bucket Policy».
- Taller guiado con dos Route Tables privadas inicialmente, RT pública añadida y Endpoint Policy Deny/Allow.
- Ejercicios abiertos de fallos: falta asociación RT, policy Deny, diferencia Gateway e Interface Endpoints.
- Criterios de elección: evitar NAT para S3, no asumir salida universal Internet.
- Fuente: transcripción suministrada «[HOL] Create a VPC Endpoint.txt» e imágenes de consola del laboratorio; analogía editorial añadida bajo AWS-ONION-LEGO.

[[VPC Gateway Endpoint - Identify Route Connect LEGO]]
[[Prefix Lists - El directorio de direcciones de la ciudad LEGO]]
[[NAT - La salida compartida de la ciudad LEGO]]
