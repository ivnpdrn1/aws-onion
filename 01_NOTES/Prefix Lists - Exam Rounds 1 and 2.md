---
type: exam-notes
project: AWS-ONION-LEGO
topic: Prefix Lists
date: 2026-11-18
---
# Prefix Lists — Exam Rounds 1 and 2

## Evaluaciones completadas y verificadas
- **Ronda 1: 10/10 (intento 1)**.
- **Ronda 2: 10/10 (intento 2 de esa ronda)**. El estado más reciente confirma diez respuestas correctas. No asumir detalles de un eventual intento anterior no documentado.
- Rendimiento actual: ambas rondas aprobadas 10/10. **No se ha evaluado aún un Nivel 2 abierto de diseño sin opciones.**

## Lo directamente enseñado en Prefix Lists.txt
- Propósito: simplificar configuración y mantenimiento de CIDR.
- AWS-managed: referencias administradas para CloudFront, S3 y DynamoDB.
- Customer-managed: el cliente define y modifica rangos.
- SG inbound: Prefix List como **Source** para permitir CloudFront a ALB.
- RT: Prefix List como **Destination** con target configurado.
- Creación: IPv4 o IPv6.

## Ampliaciones evaluadas en Ronda 2
- Bloquear el origen ALB a IP CloudFront no identifica de forma exclusiva una distribución.
- Gateway Endpoint S3/DynamoDB puede utilizar una Prefix List como destino en Route Table; el endpoint, no la lista, conecta.
- Quota de SG: peso/capacidad máxima, no garantía de una única unidad.
- APIs públicas de terceros desde EC2 privada pueden requerir NAT público u otro camino de egreso.
- Criterios de ruta: prefijo más largo y reglas de prioridad.

## Recuperación activa: VERBO→OBJETO→RESPONSABILIDAD
1. Prefix List: GROUP/REFERENCE CIDR, no conectividad.
2. SG: ALLOW fuentes (inbound) o destinos (outbound).
3. RT: ROUTE a IP destino usando Target.
4. NAT: TRANSLATE y permite conectividad de salida diseñada.
5. Endpoint: CONNECT a servicio compatible.
6. AWS-managed vs Customer-managed: diferencia es **quién mantiene**.
7. IPv4 vs IPv6: familia seleccionada al crear lista.
8. CIDR centralizado reduce **trabajo de mantenimiento**, no automáticamente todas las cuotas.
9. CloudFront Prefix List limita IP de origen; no autentica distribución.
10. Compartir una lista: cuidar efecto en todos los consumidores cuando se modifica.

## Próximo desafío abierto (NO completado)
Cinco escenarios sin opciones: CloudFront→ALB, S3/DynamoDB endpoint, SG de red corporativa, RT hacia Transit Gateway, comparar Prefix List con NAT.
Puntaje por escenario: VERBO/OBJETO 4, RESPONSABILIDAD 4, RELACIONES 4, DECISIÓN 4, FALLOS/LÍMITES 4; objetivo **20/20**.

[[Prefix Lists - Group Reference Reuse LEGO]]
