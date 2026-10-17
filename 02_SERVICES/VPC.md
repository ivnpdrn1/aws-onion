---
type: service-dna
service: "VPC"
verb: "ISOLATE"
object: "Virtual networking boundary and address space"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# VPC — ISOLATE

> [!abstract] LEGO block
> **ISOLATE → Virtual networking boundary and address space**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **ISOLATE** |
| **OBJETO** | Virtual networking boundary and address space |
| **RESPONSABILIDAD / WHY** | isolate virtual networking boundary and address space para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | Regional |
| **UPSTREAM (entrada/dependencia)** | Workload networking requirements |
| **DOWNSTREAM (salida/interfaz)** | Subnets, route tables, SG, network connections |
| **RELACIONES / COOPERATES** | EC2, IGW, NAT gateway |
| **NO ES SU TRABAJO** | Does not by itself connect to internet |
| **EXAM TRIGGER** | Custom networking, segmentation |

## Encaje LEGO
- **Puerto de entrada:** Workload networking requirements.
- **Acción central:** ISOLATE sobre Virtual networking boundary and address space.
- **Puerto de salida:** Subnets, route tables, SG, network connections.
- **Conexiones laterales:** EC2, IGW, NAT gateway.
- **Regla de descarte:** Does not by itself connect to internet.

## Pregunta de arquitectura
**¿Qué requisito exige «ISOLATE» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
