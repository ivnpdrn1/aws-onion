---
type: service-dna
project: AWS-ONION-LEGO
topic: NAT Gateway and NAT Instance
updated: 2026-11-17
tags: [aws-onion-lego, nat, igw, vpc, routing, subnet, availability, networking]
---
# NAT Gateway y NAT Instance — ROUTE → TRANSLATE → CONNECT

## 1. NECESIDAD antes de servicio
Una EC2 ubicada en una **private subnet** debe iniciar conexiones IPv4 externas (actualizar software, consumir API) y recibir sus respuestas, **sin IP pública propia ni acceso inbound nuevo directo desde Internet**. La ruta local de VPC solo cubre los rangos internos; necesitamos un intermediario que traduzca origen y gestione retornos.

**NAT** = Network Address Translation. No es sustituto de Security Group, Internet Gateway ni Application Load Balancer.

## 2. AWS-ONION-LEGO — VERBO → OBJETO → RESPONSABILIDAD → RELACIÓN
| LEGO | Verbo | Objeto | Qué hace / qué NO hace |
|---|---|---|---|
| EC2 privada | INITIATE / EXECUTE | conexión hacia API externa | Origina solicitud; no necesita public IP |
| CIDR/Private Subnet | PARTITION | IPs privadas | Segmenta VPC; no decide por sí sola el tipo de conectividad |
| Private Route Table | ROUTE | IP destino | 0.0.0.0/0 → NAT; ruta VPC local gana para tráfico interno |
| Public NAT Gateway | TRANSLATE | flujos IPv4 | Traduce y recuerda conexión iniciada; no ofrece nueva entrada Internet directamente a EC2 |
| Public Subnet + RT | HOST / ROUTE | NAT e IGW | Ubicación/ruta requerida para NAT público con salida Internet |
| Elastic IP | IDENTIFY | dirección pública fija | Es IP que observa servicio externo tras egress por NAT público |
| Internet Gateway | CONNECT | VPC a Internet | Proporciona acceso público; no reemplaza NAT |
| SG / NACL | ALLOW / FILTER | tráfico | SG stateful; NACL stateless; rutas no equivalen a permiso |
| NAT Instance | TRANSLATE / FORWARD | EC2 autogestionada | Alternativa con mantenimiento, parches, HA y scaling propios |
| Source/Destination Check | VALIDATE | tráfico de EC2 | Debe desactivarse en NAT instance para tráfico reenviado de terceros |
| Private NAT Gateway | TRANSLATE PRIVATE | tráfico entre redes | Hacia TGW/VGW/red privada; sin Elastic IP; no salida Internet por IGW |
| Multi-AZ NAT | RESIST AZ FAILURE | salida externa | Una NAT pública por AZ y rutas por AZ evita dependencia de una sola AZ |

## 3. Diagrama CANÓNICO: ida y vuelta, plano de datos
```text
EC2 PRIVATE 10.0.2.10  (sin public IPv4)
     |
     | Solicitud OUTBOUND → api.proveedor.com / IP pública
     v
PRIVATE ROUTE TABLE: 0.0.0.0/0 -> nat-gw-id
     |
     v
PUBLIC NAT GATEWAY (ubicado en PUBLIC SUBNET)
     |   traduce origen y guarda mapeo del flujo
     |   Elastic IP asociada (ej. 198.51.100.50)
     v
PUBLIC ROUTE TABLE: 0.0.0.0/0 -> IGW
     |
     v
INTERNET GATEWAY → INTERNET → API
     |
     |  API responde a Elastic IP del NAT
     v
IGW → NAT Gateway (reconoce estado) → EC2 PRIVATE

NUEVO INBOUND Internet -> NAT -> EC2 privada: NO.
Cliente Internet -> ALB público -> EC2 privada: SÍ si
rutas internas, listener, target group, SG y NACL lo permiten;
NO requiere NAT para el tramo ALB -> EC2.
```
NOTA: el NAT Gateway recibe tráfico interno encaminado hacia él; el modelo vertical describe funciones, no servidores por los que el paquete salta físicamente uno a uno.

## 4. ¿Por qué NAT público está en Public Subnet?
**Derivación causal:** Para alcanzar destinos públicos por IPv4 debe disponer de Elastic IP y conectividad de su subnet hacia IGW. Private Subnet apunta a NAT Gateway; **Public Subnet del NAT** apunta a IGW. No invertirlos. Un mismo NAT público puede dar salida a muchas EC2 privadas, que comparten una IP visible para allowlist del proveedor remoto.

## 5. NAT Gateway vs NAT Instance
| Dimensión | NAT Gateway público | NAT Instance |
|---|---|---|
| Administración | AWS lo gestiona | el cliente mantiene SO, software NAT, seguridad |
| Ubicación para salida IPv4 Internet | Public Subnet con ruta IGW | EC2 en Public Subnet con public IPv4/EIP |
| Rutas privadas | 0/0 → NAT Gateway | 0/0 → ENI o instance target configurado |
| Dirección pública | Elastic IP | EIP o IP pública |
| Source/dest check | no aplica a EC2 | **desactivar** para forward |
| SG del propio NAT | no se asigna | sí |
| Alta disponibilidad | administrada dentro de una AZ; redundar entre AZ | diseñar failover, instancias/reconfigurar rutas |
| Escalabilidad | servicio gestionado | dimensionar/monitorizar/automatizar EC2 |
| SSH/bastion, personalización | no | factible según configuración, pero separar funciones suele ser mejor |
| Port forwarding manual | no | posible como solución personalizada |

**Importante sobre AMI:** el profesor menciona buscar AMI NAT especializada. Es una indicación del material didáctico, no una garantía de AMI oficial actual. Lo esencial es Linux/EC2 configurada para forwarding/NAT y source/destination check desactivado.

## 6. Dos tipos de NAT Gateway (NO confundir)
**Public NAT Gateway:** egress IPv4 hacia Internet, alojado en Public Subnet, Elastic IP y salida IGW.

**Private NAT Gateway:** traducción de IP para conectividad a otras VPC/redes privadas a través de Transit Gateway / Virtual Private Gateway; no EIP, y tráfico dirigido a Internet Gateway se descarta. No asumir que «NAT privado» equivale a «NAT público en private subnet».

## 7. Alta disponibilidad — el error oculto
Una NAT Gateway es resiliente dentro de su AZ. Si todas las private subnets dependen de NAT-A en AZ-A, una caída de AZ-A puede dejar sin salida a EC2 que siguen saludables en AZ-B/C.

```text
    AZ-A                    AZ-B
Private EC2-A           Private EC2-B
      |                       |
RT-A 0/0 -> NAT-A       RT-B 0/0 -> NAT-B
      |                       |
Public subnet A         Public subnet B
      |                       |
         IGW de la VPC
               |
            INTERNET
```
Aislar dominios de fallo (Multi-AZ) exige también redundancia en **dependencias de conectividad**, no solo en cómputo. Se pueden automatizar cambios de rutas en caso de fallo, con RTO y complejidad diferentes.

## 8. DECISIÓN ARQUITECTÓNICA — seleccionar y descartar
- «EC2 privada consume API Internet»: Public NAT Gateway (o alternativa de egress según necesidad); **no** IGW directo sin public IP.
- «Proveedor requiere allowlist pequeña»: EIP(s) NAT Gateway; servicio externo observa esas IP públicas.
- «EC2 necesita S3/DynamoDB»: considerar **VPC Gateway Endpoints** si encajan, que evitan pasar por NAT para esos servicios.
- «Red on-prem privada necesita traducción»: Private NAT Gateway + TGW/VGW y rutas compatibles, **no** IGW.
- «Personalizar iptables/port forwarding»: NAT Instance autogestionada, evaluando mantenimiento, HA, seguridad, costo.
- «Aplicación HTTP en EC2 privada debe recibir clientes Internet»: **ALB público** hacia targets privados, no NAT.
- «Requiere HA multizona»: NAT por AZ + tablas privadas por AZ, no dependencia única.

## 9. FALLOS y diagnóstico en cinco preguntas
1. ¿Quién inicia el flujo? ¿Cuál es **destination IP**?
2. ¿Qué ruta de prefijo más largo elige la private subnet? ¿Apunta al NAT correcto?
3. ¿NAT público tiene subnet pública, salida IGW y Elastic IP? ¿Hay NACL/SG adecuados para el flujo?
4. ¿La respuesta regresa a EIP/NAT y al mapeo correcto? ¿Existe continuidad de estado?
5. Si falla una AZ, ¿qué otras subnets dependen de su NAT?

## 10. Relación con capas previas
```text
SUBNET PARTITION -> ROUTE TABLE ROUTE
  -> NAT TRANSLATE -> IGW CONNECT -> API RESPOND

intra-VPC: route local -> EC2, SIN NAT
inbound web: Route53 RESOLVE -> ALB DISTRIBUTE
  -> EC2 privada EXECUTE, SIN NAT en enlace ALB→EC2
```

## Fuente / alcance
**Fuente principal:** archivo del usuario «NAT Gateways and NAT Instances.txt», clase Digital Cloud Training (NAT públicos, NAT Instances, NAT privados, HA AZ). Los ejemplos de IP, diagnóstico, alternativas con VPC Endpoints y anotaciones de alcance son desarrollos adicionales de AWS-ONION-LEGO, no citas del profesor.

## Conexiones Obsidian
[[Subnet Types - Inbound Outbound AWS Onion LEGO]]
[[Elastic Load Balancing - Distribute LEGO Service DNA]]
[[NAT - Exam Decision Matrix and Failure Scenarios]]
[[NAT - La salida compartida de la ciudad LEGO]]
