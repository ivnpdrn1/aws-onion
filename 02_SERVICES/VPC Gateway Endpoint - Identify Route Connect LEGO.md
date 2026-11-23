---
type: service-dna
project: AWS-ONION-LEGO
topic: S3 Gateway VPC Endpoint
date: 2026-11-22
tags: [aws-onion-lego, vpc, gateway-endpoint, s3, prefix-list, route-table, endpoint-policy, cidr]
---
# S3 Gateway VPC Endpoint — IDENTIFY → ROUTE → CONNECT

## 1. NECESIDAD: ¿para qué fue creado?
Una EC2 dentro de una VPC necesita acceder a **Amazon S3** sin depender del camino general hacia Internet (IGW/NAT). Para S3 se puede crear un **Gateway VPC Endpoint**, asociar las Route Tables necesarias y utilizar una ruta cuyo destino es la **AWS-managed Prefix List de S3** y cuyo target es el **ID del endpoint**.

La pieza nueva **NO sustituye** a Route Table, Prefix List ni IAM; las conecta para habilitar una función más específica: comunicación con S3.

## 2. Diccionario inequívoco: DESTINATION ≠ TARGET
En lenguaje común «destino» y «objetivo» parecen sinónimos. En **AWS Route Table** son campos diferentes:
- **Destination = WHERE / ¿A qué IP final va el paquete?** Un CIDR o Prefix List que agrupa direcciones IP de destino.
- **Target = VIA / ¿Por qué recurso o camino se encamina?** Por ejemplo local, igw-id, nat-gw-id, vpce-id.
- El paquete mantiene como destino la IP del servicio S3. **No se sustituye la IP de destino por el ID del Gateway Endpoint**.
- Route Table **ROUTE/SELECT**; VPC Endpoint **CONNECT/PROVIDE PATH**.
- Prefix List **GROUP/IDENTIFY CIDR**: no es hop físico, no autoriza ni transporta paquetes.

La ruta real mostrada en las capturas del laboratorio (us-east-1):
| Destination | Target | Lectura humana |
|---|---|---|
| `pl-63a5400a` | `vpce-0302c9774934f0bfa` | IP incluidas en Prefix List de S3 → usar S3 Gateway Endpoint |
| `10.0.0.0/16` | `local` | IP internas de VPC → conectividad local |

En imágenes, pulsar `pl-` abre **Managed prefix lists** (`com.amazonaws.us-east-1.s3`); pulsar `vpce-` abre **Endpoints**. **Ese recorrido de navegación confirma los tipos de recursos**, no invierte las columnas.

CIDR ilustrados en la Prefix List del ejemplo: `16.182.0.0/16`, `18.34.0.0/19`, `18.34.232.0/21`, `3.5.0.0/19`, `52.216.0.0/15` y otra entrada visible parcialmente. Son entradas de una captura histórica, **no rangos vigentes garantizados**. AWS mantiene la lista regional.

## 3. Service DNA: VERBO → OBJETO → RESPONSABILIDAD → RELACIONES
| Pieza | VERBO | Responsabilidad | Lo que NO hace |
|---|---|---|---|
| EC2 / AWS CLI | REQUEST | Inicia solicitud HTTPS S3 usando credenciales/role | No configura rutas |
| DNS | RESOLVE | Resuelve hostname S3 a IP | No selecciona el camino |
| Private Subnet | CONTAIN | Aloja ENI con IP privada y está asociada a Route Table | No bloquea por sí sola el acceso S3 |
| S3 managed Prefix List | GROUP / IDENTIFY | Agrupa IP/CIDR del servicio S3 regional para matching | No proporciona camino ni autoriza |
| Route Table | ROUTE / DECIDE | Selecciona ruta según IP destino y reglas de prioridad | No construye conectividad por sí sola |
| Gateway Endpoint | CONNECT | Proporciona acceso a S3 sin NAT/IGW en ese recorrido | No es ENI privada en subnet ni NAT |
| IAM Role Policy | AUTHORIZE IDENTITY | Permite acciones S3 de la identidad EC2 | No establece ruta |
| Endpoint Policy | LIMIT ACCESS THROUGH ENDPOINT | Restringe solicitudes que usan ese endpoint | No impide accesos por otra ruta |
| Bucket Policy | CONTROL RESOURCE | Reglas del recurso S3; puede restringir a aws:SourceVpce | No sustituye route table |
| Amazon S3 | PROCESS / RESPOND | Valida solicitud y sirve/rechaza operación | No tiene que estar dentro de subnet |

## 4. Flujo end-to-end (funcional, no saltos físicos)
```text
EC2 privada — REQUEST [AWS CLI S3]
         |
DNS — RESOLVE hostname S3 → IP destino
         |
SUBNET — ROUTE TABLE ASOCIADA
         |
MATCH: IP destino pertenece a S3 Prefix List (pl-...)
         |
ROUTE TABLE — SELECT: DESTINATION pl-... → TARGET vpce-...
         |
S3 GATEWAY ENDPOINT — CONNECT vía infraestructura AWS
         |
IAM + Endpoint + Bucket Policies — AUTHORIZE / LIMIT
         |
S3 — PROCESS / RESPOND
         |
EC2 — RECEIVE respuesta HTTPS
```
**Precisión conceptual:** la Prefix List no es una estación del recorrido. La Route Table es configuración de enrutamiento, no servidor intermedio. Las políticas son evaluaciones de permisos, no cajas de red atravesadas en serie.

**DNS:** en el patrón típico de Gateway Endpoint, el nombre S3 puede resolver a direcciones IP públicas del servicio; se selecciona la ruta Prefix List→Endpoint. No implica una IP S3 privada dentro de la VPC.

## 5. Secuencia del laboratorio «[HOL] Create a VPC Endpoint.txt»
1. **VPC → VPC and more**: `lab1-custom-vpc`; 2 AZ; 2 subnets públicas + 2 privadas; NAT Gateway **None**; opción para crear S3 Gateway Endpoint.
2. **Endpoint**: `com.amazonaws.us-east-1.s3`, tipo **Gateway**; revisar **Route tables** asociadas: las dos privadas.
3. **Rutas privadas**: `pl-63a5400a → vpce-0302c9774934f0bfa` y `10.0.0.0/16 → local`.
4. **EC2 en subnet pública**: auto-assign public IPv4; IAM Role con permiso de S3 read-only (trust a EC2); conectar con EC2 Instance Connect y ejecutar `aws s3 ls`. Si no hay buckets, puede terminar sin mostrar líneas.
5. **Prueba 1**: antes de asociar tabla pública al Gateway Endpoint, la EC2 pública sale por IGW hacia S3 y puede listar si está autorizada.
6. **Prueba 2**: modificar la Endpoint Policy a Deny; la instancia pública todavía puede listar por IGW, porque **no usa** ese endpoint.
7. **Prueba 3**: asociar la **Route Table pública** al endpoint. AWS crea ruta S3 Prefix List → endpoint; ahora la política Deny causa `AccessDenied` al acceder por él.
8. **Prueba 4**: restaurar Endpoint Policy a Allow; la operación autorizada vuelve a funcionar a través del endpoint.
9. **Limpieza**: terminar EC2 y eliminar recursos de laboratorio cuando se liberen interfaces.

*Detalles de seguridad*: durante laboratorios, evitar abrir SSH a Internet sin necesidad; aplicar SG/NACL, rol y políticas mínimos necesarios. Un endpoint sin NAT solo da acceso a S3 (y no Internet general); recursos de S3 pueden exigir otras rutas/servicios para operaciones adicionales.

## 6. Contraste directo con la arquitectura previa
```text
Antes: EC2 privada → RT default → NAT público → IGW → S3
Ahora: EC2 privada → RT (pl-S3→vpce-ID) → S3 Gateway Endpoint → S3
```
Un cliente Internet → ALB público → EC2 privada es otro problema: **inbound** a aplicación; no utiliza un S3 Gateway Endpoint.

## 7. Cómo elegir
- S3/DynamoDB desde EC2 VPC: considerar **Gateway VPC Endpoint** por routing a servicio y sin dependencia de NAT para esos destinos.
- API pública de tercero: el S3 Gateway Endpoint no proporciona egress general; requiere camino apropiado.
- Interface Endpoint: usa ENI/Private IP para servicios compatibles, distinto patrón de DNS, SG y costos.
- Security Group: puede permitir egress HTTPS a rangos S3; no reemplaza IAM/bucket policy.
- Si **Endpoint Policy Deny** solo impacta al asociar la Route Table de EC2: el comportamiento evidencia el cambio de ruta y la aplicación de una política al nuevo camino.

## 8. Diagnóstico de fallo
1. ¿EC2 resuelve hostname S3? ¿Qué IP destino usa?
2. ¿Qué Route Table está **asociada a su subnet**?
3. ¿Existe la ruta Destination `pl-` S3 → Target `vpce-`?
4. ¿La Route Table selecciona la ruta adecuada, y Endpoint status es Available?
5. ¿IAM role + endpoint policy + bucket policy permiten la acción? ¿SG/NACL autorizan red?
6. ¿Hay rutas alternativas (IGW/NAT) que expliquen un acceso exitoso aunque el endpoint Deny esté vigente?

## 9. Libro: formulación memorizable
«**Destination = WHERE (Prefix List S3); Target = VIA (Gateway Endpoint)**».
«**El directorio identifica la dirección; el mapa elige la vía; la conexión a S3 existe porque el endpoint está habilitado; la autorización es una evaluación separada.**»

## Conexiones
[[Prefix Lists - Group Reference Reuse LEGO]]
[[Subnet Types - Inbound Outbound AWS Onion LEGO]]
[[NAT Gateway and NAT Instance - Translate LEGO]]
[[VPC Gateway Endpoint - Exam and Troubleshooting]]
[[VPC Gateway Endpoint - La carretera especial hacia S3]]
