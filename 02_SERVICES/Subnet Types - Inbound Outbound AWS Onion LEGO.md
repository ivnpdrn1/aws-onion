---
type: aws-onion-lego-service-dna
topic: VPC Subnet Types and Bidirectional Routing
date: 2026-11-17
tags: [aws-onion-lego, vpc, subnet, route-table, igw, nat, vpn, nacl, security-group, book]
---
# Subnet Types — INBOUND / OUTBOUND — AWS-ONION-LEGO

## NECESIDAD: ¿Por qué existen distintas subnets?
Una aplicación requiere **exposición a Internet**, servidores privados que **inician conexiones externas**, conectividad con **red corporativa** y recursos que solo se comunican **dentro de la VPC**. Los tipos *public/private/VPN-only/isolated* describen principalmente **rutas asociadas y conectividad**, no cuatro clases técnicas diferentes del recurso Subnet.

**Fuente de clase:** diapositiva «Subnet Types», con rutas: Public 10.0.0.0/16 → local y 0.0.0.0/0 → IGW; Private local y default → NAT gateway; VPN-only destino 172.31.0.0/24 → VGW; Isolated local. La ruta local del caso VPN es una presuposición de VPC que no aparece escrita en esa fila de la diapositiva. Aquí se añaden ejemplos de flujo bidireccional para explicar las relaciones.

## PIEZAS LEGO — VERBO / OBJETO / RESPONSABILIDAD / LÍMITES
| Componente | VERBO | Responsabilidad |
|---|---|---|
| VPC CIDR | DEFINE | Espacio de IP privado, ej. 10.0.0.0/16; no determina automáticamente conectividad externa |
| Subnet CIDR | PARTITION | Porción del CIDR (ej. 10.0.1.0/24); AZ y asignación de IP; no determina public/private por sí solo |
| Route Table | ROUTE | Escoge target por **IP destino** y ruta con mayor prefijo coincidente |
| Route 'local' | CONNECT | Camino entre rangos de la VPC (sujeto a filtros); no es salida Internet |
| IGW | CONNECT | Punto de acceso Internet de VPC; IPv4 público/EIP de destino o origen para conexión directa |
| Public NAT Gateway | TRANSLATE | Salidas IPv4 iniciadas desde privado; permite respuestas de conexiones establecidas, no nuevos inbound directos |
| VGW / Site-to-Site VPN | CONNECT PRIVATELY | Enlace entre VPC y red remota si rutas y seguridad en ambos extremos lo permiten |
| NACL | FILTER | Stateless por subnet: entradas y salidas/puertos de respuesta deben ser permitidos |
| Security Group | ALLOW | Stateful para ENI/recurso: controla nuevos flujos y retorno de conexiones permitidas |
| ENI / IP privada | IDENTIFY | Identifica la interfaz/endpoint dentro de VPC |
| EC2 / ALB | EXECUTE / DISTRIBUTE | Aplica la función, no reemplaza el enrutamiento ni los filtros |

## MAPA DE CUATRO SUBNETS (ejemplo didáctico IPv4)
```text
VPC 10.0.0.0/16
|
+-- PUBLIC     10.0.1.0/24  | EC2 10.0.1.10 | RT local + 0/0 -> IGW
+-- PRIVATE    10.0.2.0/24  | EC2 10.0.2.10 | RT local + 0/0 -> NAT
+-- VPN-ONLY   10.0.3.0/24  | EC2 10.0.3.10 | RT local + 172.31.0.0/24 -> VGW
+-- ISOLATED   10.0.4.0/24  | EC2 10.0.4.10 | RT local only
```
Los prefijos de subnet son ejemplos añadidos a la diapositiva, no direcciones de recursos reales.

## REGLA RECTORA
```text
PAQUETE: source IP → destination IP
                         |
             RT de la subnet del EMISOR
                         |
             coincidencia más específica
                /        |        \
             local      IGW     NAT / VGW
                         |
         Security Groups / NACL / rutas inversas
                         |
                ¿el paquete llega?
```
**INBOUND y OUTBOUND** son relativos al recurso/subnet observado. La tabla de rutas se usa para encaminar tráfico **desde la subnet**; no existe una opción «inbound/outbound» en cada fila. Para tráfico entrante desde Internet o VPN también cuentan rutas en el otro extremo, mapeo IP y controles.

## PUBLIC — conexión directa mediante IGW
RT: 10.0.0.0/16 → local; 0.0.0.0/0 → IGW.
- **Inbound nuevo de Internet:** cliente público 203.0.113.20 → IP pública/EIP de EC2 (ej. 198.51.100.10) → IGW asigna a IP privada 10.0.1.10 → SG/NACL → aplicación escuchando puerto 443.
- **Outbound:** EC2 10.0.1.10 con IP pública → RT default IGW → Internet; traducción 1:1 pública/privada del IGW.
- EC2 sin IP pública no se hace accesible directamente por el mero hecho de residir en public subnet.

## PRIVATE — NAT de salida, aplicación privada puede recibir vía ALB
RT: local y default → public NAT gateway situado en public subnet, con IGW y EIP para acceso Internet.
- **Outbound nuevo:** EC2 privada 10.0.2.10 → NAT Gateway → IGW → Internet.
- **Inbound de respuesta:** servidor externo → IGW → NAT → EC2, **solo si corresponde a flujo previamente iniciado**.
- **Nuevo inbound externo hacia NAT:** no publica la EC2; NAT no es balanceador ni proxy inbound.
- **Inbound de usuarios vía ALB:** navegador → ALB público → conexión interna VPC por ruta local hacia EC2 privada; se deben permitir SG del target desde SG del ALB, NACL y health checks. **No atraviesa NAT** en tramo ALB → EC2.

## VPN-ONLY — red remota privada
RT: local y ruta 172.31.0.0/24 → VGW; sin default hacia IGW en la diapositiva.
- **Outbound a red corporativa:** EC2 10.0.3.10 → RT → VGW → túnel VPN → 172.31.0.25.
- **Inbound desde oficina:** 172.31.0.25 → VPN/VGW → 10.0.3.10 con rutas de retorno y SG/NACL/controles acordes.
- El camino VPN es potencialmente bidireccional; no se infiere Internet directo. Las rutas del extremo corporativo son esenciales.

## ISOLATED — ruta local solamente
RT: 10.0.0.0/16 → local, no hay rutas fuera de la VPC (en este ejemplo).
- **Outbound 10.0.4.10 → 10.0.2.10:** puede ser enrutado localmente y sujeto a seguridad.
- **Inbound 10.0.1.10 → 10.0.4.10:** puede llegar localmente si seguridad y servicio lo permiten.
- **Outbound 8.8.8.8:** no existe ruta.
- **Inbound desde Internet directo:** no existe el camino directo; aislada no significa inaccesible desde las otras subnets de la VPC.

## UNA TABLA PARA EXAMEN
| Tipo | Nuevo inbound Internet directamente a EC2 | Outbound Internet | Conexión intra-VPC |
|---|---|---|---|
| Public | Sí, **si** EC2 tiene public IP, ruta IGW, permisos y servicio | Sí con IP pública y demás requisitos | Ruta local |
| Private | No mediante NAT; sí a través de intermediarios como ALB público | Sí vía NAT público bien configurado | Ruta local |
| VPN-only | No vía IGW en esquema dado | Sin salida Internet directa; sí red remota vía VPN | Ruta local |
| Isolated | No | No (esquema dado) | Ruta local |

## RELACIONES Y DECISIONES ARQUITECTÓNICAS
- Requisito «servidor web visible»: ALB público distribuido en public subnets + EC2 privada; no hace falta poner todas las EC2 públicas.
- Requisito «descargar parches desde EC2 sin IP pública»: private subnet + NAT, o endpoints/alternativas específicas según servicio.
- Requisito «acceso desde sede»: conectividad VPN/VGW y rutas de ambos extremos; no sustituye IAM ni SG.
- Requisito «DB aislada de Internet»: isolated subnet + SG restrictivo, pero la ruta local permite tráfico intra-VPC si autorizado.
- Pregunta de troubleshooting: ¿qué IP **destino** tiene el paquete?, ¿cuál es el match más específico?, ¿qué target?, ¿cómo retorna?, ¿lo permite SG/NACL?, ¿tiene IP pública cuando es necesario?

## LOS 5 CRITERIOS DEL LIBRO
1. **VERBO/OBJETO**: ROUTE vs CONNECT vs TRANSLATE vs FILTER vs EXECUTE.
2. **RESPONSABILIDAD**: RT dirige por destino, NAT traduce, SG/NACL filtran, IP identifica.
3. **RELACIONES**: ida y vuelta, IP privada vs pública, VPC local vs VPN vs Internet.
4. **DECISIÓN**: Public/Private/VPN-only/Isolated según quién inicia, alcance y riesgo.
5. **FALLOS/LÍMITES**: ruta ≠ acceso autorizado ≠ conectividad efectiva; NAT ≠ inbound; Private ≠ aislamiento total.

## Fuentes y enlaces internos
Fuente base: imagen compartida «Subnet Types» (Digital Cloud Training). Ejemplos de IP/flujo y matices son explicación arquitectónica adicional.
[[AWS-ONION-LEGO - Cuatro Bloques Secuenciales]]
[[ALB - Five Blocks Normalized ASCII AWS Onion LEGO]]
[[Subnet Types - Trafico de ida y vuelta en la ciudad LEGO]]
