---
type: concept
domain: compute-networking
tags: [aws, vpc, nat-gateway, route-table, eni, aws-onion, exam]
updated: 2026-10-13
---

# NAT Gateway — Onion privada a Internet

> **Modelo mental:** quién origina → con qué identidad → quién dirige → quién traduce → quién conecta.

## El problema
Una EC2 en subnet privada necesita descargar actualizaciones o acceder a servicios externos **sin recibir una IP pública propia**. Su dirección RFC1918 (ej. `10.0.2.10`) no puede usarse como dirección de origen enrutable en Internet.

## Functional Onion — por qué y cómo

| Eslabón lógico | ¿Por qué existe? | ¿Cómo lo implemento? | ¿Qué debo recordar? |
|---|---|---|---|
| 1. EC2 privada | Origina la conexión | Launch en private subnet sin Public IP | **Quién** |
| 2. ENI + Private IP | Identidad de red del origen | ENI primaria con `10.0.2.10` | **Con qué dirección** |
| 3. Private Route Table | Dirige destinos fuera de la VPC | `0.0.0.0/0 → nat-...` | **Hacia dónde** |
| 4. Public NAT Gateway (zonal) | Traduce origen privado; mantiene estado de conexiones | Crear NAT público con Elastic IP en subnet pública | **Quién traduce** |
| 5. Public subnet Route Table | Proporciona ruta hacia Internet | `0.0.0.0/0 → igw-...` | **Por dónde sale** |
| 6. Internet Gateway | Conecta la VPC a Internet y mapea IP privada del NAT con EIP pública | Attach IGW a VPC | **Puerta exterior** |
| 7. Internet | Destino de la solicitud | Ej. repositorio de actualizaciones | **Destino** |

**Importante:** Las Route Tables son decisiones de encaminamiento, NO dispositivos físicos atravesados uno tras otro. La ruta efectiva se evalúa en el plano de red AWS. Security Groups y NACL deben permitir el flujo.

```text
EC2 privada (10.0.2.10)
  ↓ interfaz ENI (identidad de origen)
Private RT: 0.0.0.0/0 → NAT
  ↓
NAT Gateway público zonal (ubicado en PUBLIC SUBNET)
  ↓ NAT cambia el origen a IP privada del NAT
Public RT: 0.0.0.0/0 → IGW
  ↓ IGW traduce la IP privada del NAT a su EIP pública
Internet → respuesta → traducciones inversas → EC2
```

## Comparación: qué es / qué NO es

| Elemento | Es | NO es |
|---|---|---|
| ENI | Interfaz virtual de red de un recurso | El NAT de la EC2 privada |
| Subnet | Segmento de direcciones de VPC | Un dispositivo NAT |
| Route Table | Reglas de encaminamiento asociadas a subnets | Una ENI ni una máquina |
| NAT Gateway | Recurso gestionado por AWS que hace NAT | Un setting en la ENI de EC2 |
| IGW | Gateway de la VPC a Internet | Un sustituto del NAT público |

## ¿Por qué no se puede conectar desde Internet directamente?
Un NAT Gateway público permite **conexiones iniciadas desde dentro y sus respuestas**, pero no expone la EC2 a conexiones entrantes arbitrarias. No sustituye SG/NACL ni proporciona acceso SSH directo.

## NAT Instance vs NAT Gateway
- NAT Instance: es EC2 administrada por nosotros; hay que **desactivar Source/Destination Check**, mantener software, dimensionar y crear HA.
- NAT Gateway: servicio administrado, sin SSH ni Security Group propio, con escalado y resiliencia manejados por AWS (en la opción zonal, dentro de su AZ).
- Para tolerancia a caída de AZ en diseño zonal, colocar NAT en cada AZ y dirigir cada subnet privada al NAT local. También existe la opción Regional NAT Gateway: no confundirla con el esquema zonal clásico.
- Evaluar coste por hora y tráfico procesado; considerar VPC Endpoints para servicios AWS.

## Técnica de memoria: cinco preguntas
1. **¿Quién inicia?** EC2.
2. **¿Con qué identidad?** ENI + IP privada.
3. **¿Quién decide ruta?** Route Table.
4. **¿Quién traduce?** NAT Gateway.
5. **¿Quién conecta a Internet?** IGW.

**Fórmula:** Identidad → Dirección → Traducción → Salida.

## Exam Cram
- Private subnet + salida IPv4 a Internet sin IP pública = **Public NAT Gateway + rutas adecuadas**.
- NAT zonal **en subnet pública**, EC2 origen **en subnet privada**.
- Private RT → NAT; public RT → IGW.
- NAT Instance requiere deshabilitar source/destination check.
- NAT Gateway no acepta Security Group directamente y no se administra por SSH.
- Para conectividad *privada* a ciertos servicios AWS, considerar VPC Endpoint.

## Relaciones
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - Mental Model]]
- [[AWS-Onion - Functional Onion]]
- [[AWS-Onion - Vertical & Horizontal Onion]]

## Fuentes oficiales
- https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
- https://docs.aws.amazon.com/vpc/latest/userguide/nat-gateway-scenarios.html
