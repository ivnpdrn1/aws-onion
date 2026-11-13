---
type: concept-dna
project: AWS-ONION-LEGO
concept: Subnet Mask and CIDR
verb: SEPARATE
object: IPv4 network and host bits
updated: 2026-11-13
tags: [aws-onion, lego, cidr, network, vpc, certification]
---
# Subnet Mask and CIDR — SEPARATE / EXPRESS

> **No es un servicio AWS.** Esta es una pieza conceptual fundamental que precede a [[VPC]], [[Subnet]] y [[Route Table]].

| LEGO | Contrato |
|---|---|
| **NEED** | Saber qué direcciones pertenecen al mismo rango y dimensionarlo |
| **VERB** | Máscara **SEPARATE**, CIDR **EXPRESS** |
| **OBJECT** | Bits de dirección IPv4, prefijo y parte de host |
| **RESPONSIBILITY** | Delimitar Network ID, Host ID y tamaño del bloque |
| **UPSTREAM** | Requisitos de networking, dirección y prefijo |
| **DOWNSTREAM** | VPC **ISOLATE**, Subnet **SEGMENT**, Route Table **ROUTE** |
| **RELATIONS** | [[VPC]], [[Subnet]], [[Route Table]], [[ENI]], [[Security Group]], [[NACL]] |
| **DOES NOT** | No asigna direcciones, no enruta, no autoriza, no hace subnet pública |
| **DECISION** | Prefijo según capacidad, AZ, crecimiento, no solapamiento |

## Example

`192.168.0.5` + `255.255.255.0` (= `/24`) → `192.168.0.0/24`: 24 bits de red, 8 bits de host, 256 direcciones totales.

**Cambio:** `/24` → `/25` conserva 25 bits de red y 7 bits para direcciones. 256 → 128 direcciones; AWS subnet: 251 → 123 IPv4 asignables (cinco reservadas por subnet).

## Correct LEGO sequence

```text
ADDRESS = WHERE IS AN INTERFACE?
MASK / PREFIX = WHICH NETWORK RANGE?
SUBNET = HOW TO SEGMENT VPC RANGE INTO ONE AZ?
ROUTE TABLE = WHERE TO FORWARD THE TRAFFIC?
SG / NACL = WHICH TRAFFIC IS ALLOWED?
ENI / EC2 = WHAT RECEIVES / EXECUTES?
```

## Trap

`/24` and `/25` are **network size**, not security grade, internet exposure or target host count. Máscara no equivale a Route Table ni a Security Group.

**Guía completa:** [[Subnet Mask CIDR Network ID Host ID - AWS ONION LEGO]].

**Book seed:** [[Subnet Mask - La frontera invisible de la red]].

**Fuente:** https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html
