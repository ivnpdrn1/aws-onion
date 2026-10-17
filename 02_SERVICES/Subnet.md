---
type: service-dna
service: "Subnet"
verb: "SEGMENT"
object: "VPC CIDR subrange / AZ placement"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Subnet — SEGMENT

> [!abstract] LEGO block
> **SEGMENT → VPC CIDR subrange / AZ placement**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **SEGMENT** |
| **OBJETO** | VPC CIDR subrange / AZ placement |
| **RESPONSABILIDAD / WHY** | segment vpc cidr subrange / az placement para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | One AZ |
| **UPSTREAM (entrada/dependencia)** | VPC and AZ choice |
| **DOWNSTREAM (salida/interfaz)** | Route table association, NACL, deployed resources |
| **RELACIONES / COOPERATES** | VPC, EC2, ALB |
| **NO ES SU TRABAJO** | Subnet public/private status is route-based, not just its name |
| **EXAM TRIGGER** | AZ placement and network segmentation |

## Encaje LEGO
- **Puerto de entrada:** VPC and AZ choice.
- **Acción central:** SEGMENT sobre VPC CIDR subrange / AZ placement.
- **Puerto de salida:** Route table association, NACL, deployed resources.
- **Conexiones laterales:** VPC, EC2, ALB.
- **Regla de descarte:** Subnet public/private status is route-based, not just its name.

## Pregunta de arquitectura
**¿Qué requisito exige «SEGMENT» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Networking and Connectivity]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
