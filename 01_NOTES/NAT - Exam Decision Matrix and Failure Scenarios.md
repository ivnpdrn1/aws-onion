---
type: exam-notes
project: AWS-ONION-LEGO
updated: 2026-11-17
---
# NAT — Exam Decision Matrix and Failure Scenarios

## Principio rector
**Private RT ROUTE → Public NAT TRANSLATE → IGW CONNECT**. El NAT público usa EIP y reside en Public Subnet; EC2 privadas no necesitan public IPv4.

## 8 disparadores de certificación
| Si la pregunta exige… | Razonamiento |
|---|---|
| EC2 privada inicia conexión Internet IPv4 | NAT Gateway público, ruta 0/0 en RT privada |
| Una sola IP pública visible para varias EC2 | EIP de NAT público, útil para allowlist |
| NAT Instance no reenvía paquetes | desactivar Source/Destination Check, configurar IP forwarding/NAT |
| Actualizaciones y HA gestionadas | NAT Gateway vs NAT Instance |
| NAT personalizado, SSH, port forwarding | NAT Instance con mantenimiento propio |
| Traslado de flujo privado entre redes | Private NAT + Transit/VGW, no IGW |
| AZ falla y sigue habiendo instancias saludables | NAT por AZ + rutas de cada AZ; evitar SPOF cross-AZ |
| Cliente Internet necesita acceder app privada | ALB público→EC2 privada; NAT no publica backend |

## Cuatro errores típicos
1. NAT Gateway público colocado en private subnet pese a requerir salida Internet.
2. Confundir Elastic IP del NAT con una IP pública individual de cada EC2.
3. Creer que NAT permite nuevas conexiones entrantes desde Internet.
4. Suponer que Multi-AZ EC2 automáticamente implica Multi-AZ en el egress.

## Cinco preguntas abiertas — 4 puntos por criterio, aprobar 20/20
1. Diseñe el camino completo de solicitud/retorno entre EC2 privada y API pública, identificando qué IP ve la API.
2. Compare NAT Gateway y NAT Instance por mantenimiento, HA, personalización y source/destination check.
3. Distinga Public NAT Gateway y Private NAT Gateway con el destino de red de cada uno.
4. Proponga dos AZ con EC2 privadas, NAT Gateways separados y sus Route Tables.
5. Explique por qué ALB público recibe inbound hacia servidores privados sin involucrar NAT.

Criterios: VERBO/OBJETO, RESPONSABILIDAD, RELACIONES, DECISIÓN, FALLOS/RETORNO.

[[NAT Gateway and NAT Instance - Translate LEGO]]
[[Subnet Types - Bidirectional Routing Exam Memory]]
