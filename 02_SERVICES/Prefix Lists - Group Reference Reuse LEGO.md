---
type: service-dna
project: AWS-ONION-LEGO
topic: VPC Prefix Lists
date: 2026-11-18
tags: [aws-onion-lego, vpc, prefix-list, cidr, security-group, route-table, cloudfront, s3, dynamodb]
---
# AWS VPC Prefix Lists — GROUP → REFERENCE → REUSE → MAINTAIN

## 1. NECESIDAD (por qué fueron creadas)
Un mismo conjunto de IP/CIDR debe repetirse en muchos Security Groups o Route Tables. Cuando los rangos cambian, copiarlos en múltiples reglas genera mantenimiento duplicado, riesgo de inconsistencias y errores.

**Prefix List = conjunto reutilizable de CIDR, identificado por un ID/nombre, que configuraciones compatibles pueden referenciar.** No es gateway, firewall autónomo, route table ni traductor NAT.

## 2. VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN
| LEGO | VERBO | OBJETO | RESPONSABILIDAD / límite |
|---|---|---|---|
| CIDR | DESCRIBE | rango IP | Define un bloque IPv4/IPv6 |
| Prefix List | GROUP / REFERENCE | colección CIDR | Centraliza definición, no deja pasar ni enruta paquetes por sí misma |
| Security Group | ALLOW | flujo por source/destination y puerto | Hace cumplir reglas stateful; puede usar Prefix List como rango fuente para inbound o destino para outbound |
| Route Table | ROUTE | IP destino → target | Puede usar Prefix List en Destination con target válido; no autoriza conexión |
| NAT Gateway | TRANSLATE | direcciones/flujos | Traduce egress; no reemplazado por lista |
| VPC endpoint | CONNECT | acceso a servicio | Para S3/DynamoDB Gateway endpoint usa listas gestionadas en rutas; no confundir lista con endpoint |
| NACL | FILTER | reglas por subnet | Stateless; no asumir que soporta Prefix List como argumento de regla |
| EC2 / ALB | EXECUTE / DISTRIBUTE | aplicación | Aplica la conexión, sujeto a rutas/SG/otros controles |

## 3. BASE DIRECTAMENTE EN EL ANEXO «Prefix Lists.txt»
- Prefix Lists son una característica de Amazon VPC que simplifica configuración y mantenimiento.
- Se pueden referenciar en Security Groups y Route Tables.
- Ejemplo: AWS-managed Prefix List de CloudFront, seleccionando sus entradas IP y usándola como Source de regla inbound HTTPS de SG del ALB (reduce repetir rangos).
- Otros ejemplos mencionados: Prefix Lists administradas para S3 y DynamoDB.
- Customer-managed Prefix Lists: el cliente define y mantiene los rangos.
- Durante la creación se elige la familia de direcciones IPv4 o IPv6.
- El instructor navega en VPC Console → Managed prefix lists y en Security Groups / Route Tables.

## 4. FLUJO A — SECURITY GROUP: ¿QUIÉN PUEDE ENTRAR?
```text
USUARIO → CLOUDFRONT → ALB (origin)
                           |
                      INBOUND SG
                      TCP 443, Source =
                      Prefix List CloudFront
                           |
                           v
                      TARGET GROUP → EC2
```
La Prefix List **describe orígenes IP**; la regla del SG **autoriza el puerto**. La lista no autentica qué distribución CloudFront origina tráfico: los IP pertenecen a infraestructura compartida y puede requerirse otra protección del origen. [Ampliación arquitectónica, no afirmación de la lección].

## 5. FLUJO B — ROUTE TABLE: ¿A DÓNDE VA?
```text
EC2 → destino IP 10.30.5.20
  → RT consulta destino
  → Prefix List corporativa contiene 10.20.0.0/16, 10.30.0.0/16
  → Target Transit Gateway (ruta compatible)
  → red remota, sujeto a rutas de retorno y filtros.
```
La Prefix List **describe destinos** y la Route Table **escoge un target**. Reglas de prioridad y prefijo más largo siguen aplicando; en empates y casos especiales hay prioridades AWS que se deben consultar. [Ejemplo y sutileza ampliados].

## 6. DOS MODELOS DE PROPIEDAD
| Modelo | Quién mantiene las entradas | Cuándo |
|---|---|---|
| AWS-managed | AWS | Rangos AWS de servicios compatibles, ej. CloudFront, S3, DynamoDB |
| Customer-managed | Propietario de la lista | Redes de oficinas, socios, entornos propios; centralización de CIDR |

**No equivale a un servicio universal:** cada lista tiene su conjunto específico y puede existir en ámbitos/Regiones concretos; la reutilización depende de compatibilidad y permisos.

## 7. CONTRASTES: lo que la lista NO hace
- Prefix List != Security Group: datos de direcciones vs control de tráfico.
- Prefix List != Route Table: destinos agrupados vs camino/target.
- Prefix List != NAT Gateway: representación de CIDR vs traducción.
- Prefix List != Gateway VPC Endpoint: lista de destinos S3/DynamoDB vs conectividad al servicio.
- Prefix List != identidad autenticada: CloudFront prefix list puede proteger origen por red sin demostrar propiedad de distribución.
- Prefix List != «una regla gratis»: peso/capacidad máxima afecta cuotas SG. [Ampliación].

## 8. DECISIÓN — casos
| Necesidad | Selección razonable | Por qué |
|---|---|---|
| Cinco SG repiten mismos 18 CIDR corporativos | Customer-managed Prefix List | editar el conjunto central una sola vez |
| Permitir CloudFront → ALB origin | AWS-managed CloudFront origin-facing Prefix List como SG Source | AWS mantiene rangos; requiere evaluar autenticación adicional |
| S3/DynamoDB desde private EC2 sin NAT para ese tráfico | Gateway VPC Endpoint y rutas a listas AWS administradas | Endpoint brinda conectividad; Prefix List identifica IP de servicio |
| EC2 privada consume API pública externa | NAT público u otro egress diseñado | Prefix List no ofrece conectividad |
| Rutas a muchas redes corporativas por mismo TGW | Customer-managed Prefix List como Destination | menos entradas mantenidas en tablas compatibles |

## 9. CUOTAS, VERSIONES Y SEGURIDAD [AMPLIACIÓN]
En Security Groups el impacto sobre cuota no siempre es una regla de costo «1»: AWS-managed Prefix Lists usan peso definido por AWS y Customer-managed se contabilizan según capacidad máxima reservada. El cambio de entradas requiere control operativo, pruebas y apropiada gobernanza, porque puede modificar el alcance de muchas reglas consumidoras a la vez.

## 10. CINCO PREGUNTAS AWS-ONION
1. ¿Cuál es el **conjunto de CIDR** que queremos reutilizar?
2. ¿Quién lo **mantiene**: AWS o el cliente?
3. ¿Dónde se **referencia**: SG Source / Destination o RT Destination?
4. ¿Quién toma la **decisión operativa**: SG ALLOW / RT ROUTE / Endpoint CONNECT?
5. ¿Qué riesgos quedan: identidad de CloudFront, cuotas, permisos, cambios de rangos, rutas retorno?

## PUENTE CON EL TODO
```text
NECESIDAD: muchos CIDR repetidos
         |
         v
PREFIX LIST — GROUP / REFERENCE
     /                     \
    v                       v
SECURITY GROUP — ALLOW     ROUTE TABLE — ROUTE
    |                       |
  ALB/EC2                 TGW/Endpoint/otro Target
```
No es una secuencia de tránsito: Prefix List es un recurso de configuración compartida que otros servicios consultan.

## Links del Vault
[[Subnet Types - Inbound Outbound AWS Onion LEGO]]
[[NAT Gateway and NAT Instance - Translate LEGO]]
[[Prefix Lists - Exam Rounds 1 and 2]]
[[Prefix Lists - El directorio de direcciones de la ciudad LEGO]]
