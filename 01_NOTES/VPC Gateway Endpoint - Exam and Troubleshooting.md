---
type: exam-notes
project: AWS-ONION-LEGO
date: 2026-11-22
topic: VPC Gateway Endpoint for S3
---
# VPC Gateway Endpoint — Exam and Troubleshooting

## Idea esencial de certificación
**EC2 REQUEST → DNS RESOLVE → Route Table ROUTE con Destination S3 Prefix List → Target S3 Gateway Endpoint CONNECT → políticas AUTHORIZE → S3 RESPOND**.

## La confusión resuelta con evidencia en la consola
- **DESTINATION: `pl-63a5400a` = Prefix List `com.amazonaws.us-east-1.s3`**. Al hacer clic en el prefijo `pl-`, la consola abre entradas CIDR S3.
- **TARGET: `vpce-0302c9774934f0bfa` = S3 Gateway Endpoint**. Al hacer clic en `vpce-`, la consola abre el recurso Endpoints.
- **`10.0.0.0/16 → local`** = tráfico intravpc.
- No invertir Destination y Target. «Destination/Target» suenan similares en lenguaje normal pero tienen semánticas distintas en las rutas.
- El paquete conserva como IP destino la de S3; `vpce-` es **target lógico** no una sustitución del destino IP.

## Preguntas de razonamiento
1. Si existe Endpoint pero no está asociada la Route Table de la EC2, ¿qué ruta toma S3? (Depende de rutas alternativas.)
2. Si se aplica Endpoint Policy Deny pero `aws s3 ls` funciona, ¿qué verificar primero? (Asociación de Route Table/camino efectivo.)
3. Si `pl-S3` coincide, ¿qué recurso permite tráfico hacia S3? (El target `vpce-`.)
4. Si un IAM Role permite lectura, ¿puede Endpoint Policy Deny bloquearla? (Sí para solicitudes por el endpoint.)
5. Si la subnet no tiene NAT, ¿el Gateway Endpoint brinda Internet general? (No: acceso S3 del servicio.)
6. ¿Es el Gateway Endpoint una ENI privada que se instala en la subnet? (No; es la diferencia con Interface Endpoint.)

## Criterios 20/20 para futuras respuestas abiertas
VERBO/OBJETO, RESPONSABILIDAD, RELACIONES, DECISIÓN, FALLOS/LÍMITES. No asignar puntuaciones de evaluación no realizada.

## Lo visto directamente en el laboratorio
VPC and more, 2 AZ, 2 public+2 private, NAT None, Gateway Endpoint S3 creado, 2 RT privadas asociadas, EC2 de prueba pública con rol S3 read only, Endpoint Policy Allow→Deny, asociación RT pública, AccessDenied, restauración Allow.

## Precaución técnica
La Route Table selecciona el target para el destino IP; la Prefix List no es salto de red; las políticas no son saltos físicos. Los IDs del vídeo/captura son ilustrativos y no deben tratarse como recursos activos.

[[VPC Gateway Endpoint - Identify Route Connect LEGO]]
[[Prefix Lists - Exam Rounds 1 and 2]]
