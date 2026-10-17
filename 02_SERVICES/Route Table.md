---
type: service-dna
service: "Route Table"
verb: "ROUTE"
object: "Destination prefixes to next hops"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Route Table — ROUTE

> [!abstract] LEGO block
> **ROUTE → Destination prefixes to next hops**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **ROUTE** |
| **OBJETO** | Destination prefixes to next hops |
| **RESPONSABILIDAD / WHY** | route destination prefixes to next hops para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | VPC route table; associated subnets |
| **UPSTREAM (entrada/dependencia)** | Traffic destination |
| **DOWNSTREAM (salida/interfaz)** | Local VPC path, IGW, NAT gateway, etc |
| **RELACIONES / COOPERATES** | VPC, subnet, IGW, NAT |
| **NO ES SU TRABAJO** | Does not authorize access; is not a firewall |
| **EXAM TRIGGER** | Where should packets go? |

## Encaje LEGO
- **Puerto de entrada:** Traffic destination.
- **Acción central:** ROUTE sobre Destination prefixes to next hops.
- **Puerto de salida:** Local VPC path, IGW, NAT gateway, etc.
- **Conexiones laterales:** VPC, subnet, IGW, NAT.
- **Regla de descarte:** Does not authorize access; is not a firewall.

## Pregunta de arquitectura
**¿Qué requisito exige «ROUTE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
