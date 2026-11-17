---
type: exam-notes
topic: VPC Subnet Types
date: 2026-11-17
---
# Subnet Types — diagnóstico de flujo INBOUND y OUTBOUND

**Principio:** la ruta depende de la **IP de destino** y del longest-prefix match; la IP/CIDR de la subnet no determina sola public/private.

## Cuatro consultas mentales por paquete
1. **Origen y destino IP:** ¿quién inició el flujo?, ¿qué IP es el target final?
2. **Route Table asociada a subnet de origen:** ¿local, default IGW, NAT, ruta específica VPN o sin ruta?
3. **Camino de retorno:** ¿IGW y public IPv4, NAT con estado, VPN con rutas remotas o respuesta local?
4. **Seguridad y aplicación:** ¿SG stateful, NACL stateless, escucha puerto correcto y permisos en ambos sentidos?

## Contrastes esenciales
- Public: ruta 0.0.0.0/0 → IGW **y** IP pública de EC2 para conexión directa IPv4.
- Private: salida mediante NAT no habilita nuevas sesiones entrantes de Internet; ALB público conecta con target privado vía red local.
- VPN-only: VGW hacia CIDR corporativo; acceso bidireccional requiere rutas y permisos recíprocos.
- Isolated: no ruta fuera de VPC; sí puede comunicarse intra-VPC.
- Security Groups **stateful**, NACL **stateless**.
- Route Table no es firewall y Destination no es IP de origen.
- CIDR 10.0.0.0/16 local es más específico que 0.0.0.0/0 y gana.

## Preguntas de examen abierto (20/20)
1. Explique por qué una EC2 en subnet pública sin public IPv4 no acepta una conexión directa de Internet.
2. Dibuje ida y retorno de EC2 privada hacia repositorio externo con NAT Gateway público.
3. Dibuje cliente Internet → ALB público → EC2 privada, sin pasar por NAT.
4. ¿Cómo comunicarse desde 172.31.0.25 (corporativo) con 10.0.3.10 (AWS) a través de VPN?
5. ¿Qué puede y qué no puede alcanzar una EC2 aislada con solo ruta local?

Criterios: VERBO/OBJETO, RESPONSABILIDAD, RELACIONES, DECISIÓN, FALLOS/RETORNO.
[[Subnet Types - Inbound Outbound AWS Onion LEGO]]
