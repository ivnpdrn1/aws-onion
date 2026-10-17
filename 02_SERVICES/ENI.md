---
type: service-dna
service: "ENI"
verb: "ATTACH"
object: "Virtual network interface and addresses"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# ENI — ATTACH

> [!abstract] LEGO block
> **ATTACH → Virtual network interface and addresses**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **ATTACH** |
| **OBJETO** | Virtual network interface and addresses |
| **RESPONSABILIDAD / WHY** | attach virtual network interface and addresses para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | AZ-bound interface |
| **UPSTREAM (entrada/dependencia)** | EC2/VPC networking configuration |
| **DOWNSTREAM (salida/interfaz)** | IP traffic to instance |
| **RELACIONES / COOPERATES** | EC2, SG, private IP |
| **NO ES SU TRABAJO** | Not a routing device or permissions identity |
| **EXAM TRIGGER** | Virtual interface, addresses, attachment |

## Encaje LEGO
- **Puerto de entrada:** EC2/VPC networking configuration.
- **Acción central:** ATTACH sobre Virtual network interface and addresses.
- **Puerto de salida:** IP traffic to instance.
- **Conexiones laterales:** EC2, SG, private IP.
- **Regla de descarte:** Not a routing device or permissions identity.

## Pregunta de arquitectura
**¿Qué requisito exige «ATTACH» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
