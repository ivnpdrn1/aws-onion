---
type: service-dna
project: AWS-Onion
topic: AWS Control Tower
date: 2026-11-11
tags: [aws-onion-lego, control-tower, landing-zone, governance, account-factory, guardrails]
---
# AWS Control Tower — ORCHESTRATE / GOVERN / STANDARDIZE

## 1. NECESIDAD — por qué existe
Una empresa comienza con una cuenta, luego separa Development y Production en cuentas miembro mediante AWS Organizations, OU y SCP. Con decenas de cuentas, la organización necesita **aprovisionamiento repetible, registro centralizado, auditoría, acceso coordinado y controles continuos**. Repetir manualmente las configuraciones introduce diferencias y errores.

**AWS Control Tower** fue creado para orquestar una **landing zone**: una base multicuentas gobernada, preparada con convenciones de seguridad y operación. No sustituye AWS Organizations: se apoya en él y en otros servicios.

## 2. LEGO: VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN
| Pieza | Verbo / objeto | Responsabilidad y límite |
|---|---|---|
| Organizations | ORGANIZE / cuentas | jerarquía Root-OU-Account, SCP y administración multiaccount; no es landing zone completa por sí solo |
| SCP | LIMIT / acciones | máximo organizativo en cuentas miembro; no concede permiso IAM |
| IAM | AUTHORIZE / principal | permisos de una identidad y roles, sujetos a políticas aplicables |
| IAM Identity Center | FEDERATE / ASSIGN / accesos | acceso de usuarios a cuentas con permission sets; origen de identidades puede integrarse/federarse con IdP |
| Control Tower | ORCHESTRATE / landing zone | coordina configuración y gobierno multicuentas; no ejecuta aplicaciones empresariales |
| Landing Zone | STANDARDIZE / entorno | baseline organizativo: cuentas compartidas, logging, acceso, controles |
| Account Factory | PROVISION / cuentas | crea cuentas gobernadas repetiblemente; no reemplaza Organizations |
| Management Account | MANAGE / organización | base administrativa y facturación, no alojamiento recomendado de workloads regulares |
| Log Archive Account | PRESERVE / logs | almacenamiento central de registros de CloudTrail y Config; no es principalmente equipo de auditoría |
| Audit Account | INSPECT / seguridad | operaciones de auditoría y acceso de seguridad entre cuentas, no almacén principal de logs |
| AWS CloudTrail | RECORD / API events | registro de actividad admitida, no evalúa por sí mismo conformidad de configuraciones |
| AWS Config | EVALUATE / recursos | evalúa configuraciones contra reglas, no impide automáticamente una operación |
| Control preventivo | PREVENT / acciones | restricciones vía políticas Organizations (por ejemplo SCP y otros tipos según control) |
| Control detective | DETECT / incumplimiento | evaluación vía Config; detección no equivale a remediación |
| Control proactivo | VALIDATE / recurso | comprobación durante aprovisionamiento compatible mediante CloudFormation Hooks |

## 3. RELACIONES en dos planos (no confundir una jerarquía con un flujo HTTP)
```text
                  AWS CONTROL TOWER
                  ORCHESTRATE / GOVERN
                           |
                  AWS ORGANIZATIONS
                      Root / OU
             +-------------+-------------+
             |             |             |
         Security OU    Sandbox OU     Production OU
             |             |             |
       +-----+-----+     Dev/Test         Prod
       |           |
   Log Archive    Audit
    PRESERVE     INSPECT
             |
      Management Account (bajo Root)
             |
       IAM Identity Center
          ACCESS / FEDERATE

Control Tower -> Account Factory -> cuentas configuradas
Control Tower -> controls -> Prevent / Detect / Proactive
CloudTrail -> Log Archive     Config -> evaluations
```
**Diagrama conceptual del curso**: Root, Security OU (Audit y Log Archive), Sandbox OU (Dev/Test), Production OU (Prod), Management Account. La ubicación y requisitos exactos de OU dependen de la versión y configuración de la landing zone.

## 4. SECUENCIA causal
1. **SEPARATE**: cuentas para aislar producción, pruebas y seguridad.
2. **ORGANIZE**: Organizations crea jerarquía y límites SCP.
3. **STANDARDIZE**: Control Tower configura la landing zone y cuentas compartidas.
4. **ACCESS**: Identity Center permite asignar acceso centralizado a cuentas.
5. **PROVISION**: Account Factory repite una plantilla gobernada para nuevas cuentas.
6. **RECORD/PRESERVE**: CloudTrail/Config proporcionan eventos y datos de configuración, Log Archive conserva registros.
7. **INSPECT/EVALUATE**: Audit y Config ayudan a verificar seguridad y cumplimiento.
8. **PREVENT/DETECT/VALIDATE**: controles bloquean, señalan o validan según momento y tipo; remediar es una acción adicional.

## 5. Arquitectura y alternativas
| Si necesito… | Seleccionar | Por qué no elegir solo… |
|---|---|---|
| Agrupar cuentas y políticas | Organizations + OU/SCP | Identity Center gestiona acceso, no organiza OU |
| Landing zone estandarizada | Control Tower | Organizations solo no automatiza toda esa base |
| Aprovisionar cuentas repetibles | Account Factory | Create account manual exige reconfiguración |
| Acceso centralizado a cuentas | IAM Identity Center | SCP limita; no asigna acceso de usuarios |
| Conservación separada de evidencias | Log Archive + CloudTrail/Config | Audit es función de examen/operaciones, no archivo principal |
| Detectar bucket S3 no conforme | Config / detective | CloudTrail registra eventos, no sustituye reglas de configuración |
| Bloquear API prohibida | preventive / SCP aplicable | detective identifica después y no bloquea |
| Validar durante CloudFormation | proactive / CloudFormation Hooks | detective evalúa cumplimiento de configuraciones existentes |

**Alternativa válida**: construir una landing zone propia mediante Organizations, IAM Identity Center, Config, CloudTrail, infraestructura como código y automatización; más libertad y esfuerzo de mantenimiento. Control Tower incorpora reglas, recursos gestionados y convenciones que deben respetarse.

## 6. Matices para precisión técnica y el libro
- La **Management Account** no da acceso universal automático a cualquier principal de otras cuentas; se requieren mecanismos cross-account autorizados.
- El **root user** no es la práctica habitual para administrar cuentas.
- **Audit ≠ Log Archive**: inspeccionar frente a conservar evidencia.
- **Detective ≠ automatic remediation**; remediation debe ser diseñada.
- En las diapositivas aparecen **preventive y detective**; añadimos proactivos como ampliación arquitectónica del estudio.
- Los controles concretos y las OU requeridas pueden cambiar entre versiones; no memorizar un dibujo como condición universal.
- Este tema trata el **control plane de gobierno**, no sustituye el data plane: Route 53/ALB/EC2.

## 7. Puentes de estudio
[[AWS Organizations - Organize Govern Assume LEGO Service DNA]]
[[SCP Strategies and Inheritance - Architecture LEGO]]
[[AWS Control Tower - Exam Memory and Round 1]]
[[AWS Control Tower - El arquitecto de los edificios]]
