---
type: service-dna
service: "Nitro System"
verb: "OFFLOAD"
object: "Hardware-assisted virtualization and I/O"
updated: 2026-10-16
tags: [aws-onion, service-dna, lego, certification]
---
# Nitro System — OFFLOAD

> [!abstract] LEGO block
> **OFFLOAD → Hardware-assisted virtualization and I/O**

| Contrato | Respuesta |
|---|---|
| **VERBO** | **OFFLOAD** |
| **OBJETO** | Hardware-assisted virtualization and I/O |
| **RESPONSABILIDAD / WHY** | offload hardware-assisted virtualization and i/o para resolver la necesidad arquitectónica correspondiente |
| **SCOPE** | EC2 platform / instance family dependent |
| **UPSTREAM (entrada/dependencia)** | EC2 instance hardware architecture |
| **DOWNSTREAM (salida/interfaz)** | Virtualized compute, storage, network capabilities |
| **RELACIONES / COOPERATES** | EC2, ENA, EBS |
| **NO ES SU TRABAJO** | Not a VPC routing component |
| **EXAM TRIGGER** | Enhanced EC2 virtualization/security |

## Encaje LEGO
- **Puerto de entrada:** EC2 instance hardware architecture.
- **Acción central:** OFFLOAD sobre Hardware-assisted virtualization and I/O.
- **Puerto de salida:** Virtualized compute, storage, network capabilities.
- **Conexiones laterales:** EC2, ENA, EBS.
- **Regla de descarte:** Not a VPC routing component.

## Pregunta de arquitectura
**¿Qué requisito exige «OFFLOAD» y cuáles dependencias han de existir antes?** Comprobar ámbito, seguridad, disponibilidad, costes y datos antes de implementar.

## Nota desarrollada
- [[EC2 - Nitro System and Nitro Enclaves]]
- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
