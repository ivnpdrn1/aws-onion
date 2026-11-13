---
type: conceptual-guide
project: AWS-ONION-LEGO
topic: subnet-mask-cidr-ipv4
status: validated
updated: 2026-11-13
tags: [aws-onion, lego, networking, subnet-mask, cidr, vpc, certification, book-source]
---
# Subnet Mask, CIDR, Network ID y Host ID — AWS-ONION-LEGO

> **Idea maestra:** la dirección IPv4 **IDENTIFY** una dirección; la máscara **SEPARATE** los bits de red y de host; CIDR **EXPRESS** ese límite; la subnet **SEGMENT** un rango dentro de la VPC; la Route Table **ROUTE** destinos. **Route ≠ Address ≠ Permission**.

## 1. Necesidad antes de terminología

Una IP como `192.168.0.5` es insuficiente para decidir qué prefijo representa la red. La máscara `255.255.255.0` o el sufijo `/24` establece que 24 de sus 32 bits son de red y quedan 8 bits para identificar direcciones dentro de ese bloque.

**Subnet mask** no es un servicio administrado de AWS: es una herramienta matemática del direccionamiento IPv4. En AWS los bloques de VPC/subnets se expresan en notación CIDR.

## 2. Service/Concept DNA — pieza LEGO

| Dimensión | Respuesta |
|---|---|
| **NECESIDAD** | Distinguir prefijo de red y parte de host; dimensionar bloques IP |
| **VERBO** | **SEPARATE / DELIMIT** |
| **OBJETO** | Los 32 bits de una dirección IPv4 |
| **RESPONSABILIDAD** | Definir la longitud del prefijo de red y el espacio restante de direcciones |
| **ENTRADA** | Dirección IPv4 + máscara / prefijo CIDR |
| **SALIDA** | Network address, rango y tamaño del bloque |
| **RELACIONES** | IPv4 address, CIDR, VPC, Subnet, Route Table, ENI |
| **NO HACE** | No asigna IP, no habilita Internet, no concede permisos ni sustituye Route Tables/SG/NACL |
| **DECISIÓN** | Elegir un CIDR con capacidad, separación, crecimiento y no solapamiento adecuados |

## 3. Ejemplo desde la diapositiva

```text
IP                         192.168.0.5
Subnet mask                255.255.255.0
CIDR                       192.168.0.0/24
Network ID / network       192.168.0.0
Host bits                  8
Host portion of .5         00000101 (valor 5)
Total de direcciones       2^8 = 256
```

La máscara tiene **24 unos contiguos** y **8 ceros**:

```text
IP      11000000.10101000.00000000.00000101
MASK    11111111.11111111.11111111.00000000
AND     11000000.10101000.00000000.00000000
                                               => 192.168.0.0
```

La operación AND conserva los bits de la IP donde la máscara contiene 1 y produce 0 donde la máscara contiene 0. No es una «lista de direcciones» ni una política de seguridad.

## 4. ¿Qué cambia de /24 a /25?

| Propiedad | /24 | /25 |
|---|---:|---:|
| Máscara decimal | 255.255.255.0 | 255.255.255.128 |
| Bits de red | 24 | 25 |
| Bits de host | 8 | 7 |
| Direcciones totales por bloque | 256 | 128 |
| Hosts convencionales en una subred IPv4 típica | 254 | 126 |
| IPv4 disponibles para asignación en **AWS subnet** | **251** | **123** |

**Conclusión:** aumentar de `/24` a `/25` hace **más pequeño** el bloque: se toma un bit adicional para prefijo de red y las direcciones totales se reducen a la mitad. No confundir «prefijo más largo» con «más direcciones disponibles».

```text
192.168.0.0/24 (192.168.0.0 - 192.168.0.255)
  ├── 192.168.0.0/25    (192.168.0.0   - 192.168.0.127)
  └── 192.168.0.128/25  (192.168.0.128 - 192.168.0.255)
```

**AWS reserva 5 IPv4 en cada subnet** (las primeras cuatro y la última), no solo las dos convencionales de red y broadcast. Por ejemplo, para `10.0.1.0/24`: `.0` network, `.1` router VPC, `.2` reservado para DNS de AWS, `.3` uso futuro, `.255` último/broadcast reservado. Con CIDRs secundarios el DNS VPC puede estar en la dirección base de la VPC primaria + 2; evitar generalizaciones sobre cada `.2` como endpoint DNS real.

## 5. Conectar la pieza a la arquitectura de AWS

```text
REQUISITO: espacios IP aislados y escalables
       |
       v
VPC       ISOLATE (10.0.0.0/16, límite regional)
       |
       v
CIDR      EXPRESS  (prefix /16, subnet prefixes /24)
       |
       v
Subnet    SEGMENT  (10.0.1.0/24, AZ-a; 10.0.2.0/24, AZ-b)
       |
       +--> Route Table ROUTE destinos; VPC CIDR -> local
       +--> NACL FILTER en frontera de subnet (stateless)
       +--> SG AUTHORIZE tráfico a recurso/ENI (stateful)
       +--> ENI CONNECT y contiene IP privada
       +--> EC2 EXECUTE aplicaciones
```

- La **VPC es regional** y una subnet existe **en una sola Availability Zone**.
- El CIDR de cada subnet debe estar contenido en el/los CIDR de la VPC y no superponerse con otra subnet.
- Una subnet **no** es pública por usar `/24`, un nombre «public» ni por tener espacio IP amplio: su ruta hacia un Internet Gateway es el criterio arquitectónico de *public subnet*; una instancia también necesita los demás requisitos para conectividad directa.
- Un equipo IP utiliza su máscara para reconocer destinos en su enlace/subred; en **AWS**, el tráfico entre subnets sigue las rutas locales de VPC sujetas a controles de red.
- La máscara delimita **a quién pertenece la dirección**, la tabla de rutas decide **por dónde viajar**, SG/NACL determinan **qué tráfico se permite**.

## 6. Decisión arquitectónica / límites

**Caso:** una app requiere ~180 direcciones IPv4 asignables en una subnet. En AWS `/25` ofrece solo 123: **insuficiente**. `/24` ofrece 251: **suficiente** (antes de reservas de crecimiento). La selección debe considerar aumento de instancias, interfaces, servicios que consumen IP, límites de VPC, conectividad on-premises y ausencia de solapamientos.

No extrapolar esta aritmética IPv4 directamente a IPv6: su prefijado y reglas son distintos.

## 7. Diagnóstico y disparadores de examen

- `192.168.0.5/24` -> red `192.168.0.0`, 256 direcciones totales.
- `192.168.0.5/25` -> red `192.168.0.0`, 128 direcciones en el bloque.
- `192.168.0.200/25` -> red `192.168.0.128`.
- `/24` vs `/25`: **251 vs 123** disponibles en una *subnet AWS*.
- «Need two subnets in distinct AZs» -> subnets distintas, CIDRs no solapados, una AZ por subnet.
- «Need Internet access» -> subnet mask no sirve: revisar route table, gateway, IP, SG/NACL.
- «Need permission» -> no confundir máscara CIDR con Security Group ni NACL.

## 8. Preguntas para reconstruir, no memorizar

1. ¿Qué significa que `/24` use 24 bits de red? **Respuesta:** ocho bits restantes para direcciones dentro del bloque.
2. ¿Qué pasa si cambiamos `192.168.0.0/24` a `192.168.0.0/25`? **Respuesta:** 256 → 128 direcciones; subred más pequeña.
3. ¿Por qué una IP privada en una subnet no implica acceso a Internet? **Respuesta:** direccionamiento, rutas, gateway y permisos son responsabilidades diferentes.
4. ¿Qué elegirías para 180 IPv4 asignables en AWS, `/24` o `/25`? **Respuesta:** `/24`, porque 251 >= 180 y 123 < 180.

## 9. Enlaces LEGO

- [[Subnet]]
- [[VPC]]
- [[Route Table]]
- [[Security Group]]
- [[NACL]]
- [[ENI]]
- [[EC2 - Networking and Connectivity]]
- [[Subnet Mask and CIDR - Define Network Boundary LEGO]]
- [[Subnet Mask - La frontera invisible de la red]]

## Referencias oficiales AWS
- [Subnet CIDR blocks](https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html)
- [VPC CIDR blocks](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-cidr-blocks.html)
- [IP addressing for VPCs and subnets](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-ip-addressing.html)
