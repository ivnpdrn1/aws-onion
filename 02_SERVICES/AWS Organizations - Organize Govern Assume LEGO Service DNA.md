---
type: service-dna
project: AWS-Onion
topic: AWS Organizations
updated: 2026-11-09
tags: [aws-onion-lego, organizations, ou, scp, iam, sts, multi-account]
---
# AWS Organizations — ORGANIZE → GROUP → GOVERN → ASSUME → ADMINISTER

## 0. La necesidad, antes de los nombres de los servicios
**Problema inicial:** una empresa opera desarrollo y producción dentro de una cuenta. Crece y necesita cuentas separadas para aislar responsabilidades, recursos y administración sin perder gobierno central.

**Decisión:** AWS Organizations establece una jerarquía con una Management Account, un Root organizativo, OUs y cuentas miembro. IAM y STS resuelven después la administración autorizada dentro de esas cuentas.

**La clase [HOL] Create Organization and add Account.txt muestra** crear la organización desde una cuenta existente, habilitar SCP, examinar FullAWSAccess, crear una cuenta de producción con correo único, el rol OrganizationAccountAccessRole y Switch Role. **Ampliaciones de análisis** en estas notas: distinguir máximos SCP de autorizaciones IAM, evaluación del rol, y el diseño multiaccount de producción/desarrollo.

## 1. LEGO DNA (VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN)
| LEGO | VERBO y OBJETO | RESPONSABILIDAD (qué sí / qué no) | DECISIÓN |
|---|---|---|---|
| AWS Organizations | ORGANIZE cuentas | gobierno de múltiples cuentas; no reemplaza IAM de cada cuenta | usar para gobierno multiaccount |
| Management Account | MANAGE organización | administra funciones centrales; no es OU ni otro root user | separar gobierno de workloads |
| Organizations Root | ANCHOR jerarquía | nodo superior de OUs/cuentas; distinto del root user de una cuenta | herencia organizativa |
| OU | GROUP cuentas y OUs | agrupa destinos de política; no es un IAM Group | agrupar por entorno, control, área |
| Member Account | ISOLATE ámbito | frontera de recursos/identidad/facturación lógica; no hereda IAM users de otras cuentas | separar producción y dev |
| SCP | LIMIT permisos máximos | restringe principales de cuentas miembro aplicables; NO concede IAM permissions | límites centralizados |
| IAM Policy | AUTHORIZE principales | concede/deniega acciones a usuarios/roles según evaluación completa; no organiza cuentas | permisos concretos |
| IAM Role | DEFINE identidad asumible | tiene trust policy y permissions; no transforma cuenta miembro en management | operar entre cuentas sin root |
| STS AssumeRole | ASSUME identidad temporal | entrega credenciales de sesión, sujetas a controles; no cambia credenciales IAM permanentes | acceso temporal cross-account |
| Switch Role | SWITCH consola | interfaz para activar una sesión de rol; no anula permisos SCP/IAM | acceder a otra cuenta |
| OrganizationAccountAccessRole | ADMINISTER cuenta nueva | rol configurado en el proceso de creación de cuenta desde Organizations; ejemplo de AdministratorAccess | administración inicial, revisar least privilege |

## 2. Dos ONION independientes, conectadas
**ONION de gobierno (no es recorrido de red):**
```text
              AWS ORGANIZATIONS
                    |
            MANAGEMENT ACCOUNT
                    |
           ORGANIZATIONS ROOT
               /          \
         OU Production   OU Development
             |                |
        Account PROD      Account DEV
             |                |
      Recursos e IAM      Recursos e IAM

       SCP @ Root / OU / Account
          = límite, NO permiso
```
**ONION de acceso a Production:**
```text
Principal autorizado en cuenta de origen
          |
   sts:AssumeRole + trust policy compatible
          |
          v
OrganizationAccountAccessRole en PROD
          |
      AWS STS entrega
    credenciales temporales
          |
          v
  sesión de rol en PROD
          |
       políticas IAM
     + límites aplicables
          |
          v
       API / recursos
```
**La OU y SCP no son saltos de un request HTTP.** Esta es la ONION del *control plane* organizativo/identidad. Se relaciona con la ONION de networking porque AWS aplica autorizaciones a quien configura redes, ALB, EC2, etc.

## 3. Secuencia de la práctica (causa → acción → consecuencia)
1. Ya existe una cuenta AWS. Desde ella, **Create organization**: pasa a ser Management Account.
2. Se observa el Root de Organizations y la jerarquía. **Root organizativo ≠ root user**.
3. Se habilitan Service Control Policies. FullAWSAccess es la SCP permisiva predeterminada: **no es AdministratorAccess de IAM**.
4. Para separar Production, **Add AWS account**: crear nueva o invitar existente (esta última requiere aceptación).
5. Al crear nueva cuenta, proporcionar nombre y **correo exclusivo** de cuenta. Si el correo ya se utiliza, falla la creación.
6. Durante la creación, el rol de administración de ejemplo se denomina **OrganizationAccountAccessRole**; el instructor conserva el nombre predeterminado y comprueba su AdministratorAccess.
7. Tras crear, conservar Account ID, Role Name y display name. Switch Role desde un principal autorizado; la acción se basa en AssumeRole y STS.
8. La consola Organizations puede mostrar AccessDenied dentro de la cuenta miembro aunque EC2/IAM funcionen: el ámbito de administración organizativa no equivale a la administración de la cuenta.
9. Switch Back vuelve al contexto original.

## 4. Tres diferencias que no debemos confundir
### SCP LIMIT vs IAM AUTHORIZE
```text
SCP aplicable permite acción? --NO--> DENY
          |
         SÍ
          v
IAM y otras políticas autorizan? --NO--> DENY
          |
         SÍ
          v
PERMIT (si no existe otra denegación/restricción aplicable)
```
La SCP FullAWSAccess ofrece un máximo permisivo, no concede AdministratorAccess. AdministratorAccess de IAM tampoco supera un Deny explícito de una SCP aplicable. **Las SCP no restringen directamente los principales de la Management Account** (aclaración de alcance, no ejemplo de la práctica).

### OU ≠ IAM Group
Una OU agrupa **cuentas**. Un IAM Group agrupa **usuarios IAM** dentro de una cuenta. Mover una cuenta entre OUs cambia políticas organizativas aplicables, no copia sus usuarios/roles ni migra EC2.

### Organizations Root ≠ Root User
Root organizativo: nodo jerárquico superior. Root user: identidad privilegiada propia de una cuenta. Switch Role no utiliza las credenciales root de la cuenta destino.

## 5. Decisiones y descartes
| Requisito | Elegir | Descartar y razón |
|---|---|---|
| Gobernar dev y prod separadas | Organizations + cuentas miembro | IAM Groups no crean separación entre cuentas |
| Políticas comunes sobre Production | OU + SCP | IAM policy individual no establece automáticamente límite sobre todas las cuentas |
| Autorizar operación concreta en Production | IAM policies/role | SCP FullAWSAccess no concede autorizaciones |
| Administrar PROD desde cuenta origen | Role asumible + STS, confianza y permisos | compartir root user es innecesario y riesgoso |
| Incorporar cuenta externa | invitación desde Organizations + aceptación | crear OU no invita ni crea cuentas |
| Operación reservada a Management Account | usar identidad autorizada en Management | AdministratorAccess en cuenta miembro no basta |

## 6. Error de certificación frecuente
**«Como FullAWSAccess dice Allow *, todos pueden hacer todo» es falso.** SCP es límite; IAM debe autorizar. **«AssumeRole es igual a IAM Group» es falso:** el grupo mantiene identidad, el rol crea sesión temporal con otra identidad.

## 7. Para el libro
Metáfora «Una empresa de edificios»: Organizations coordina varios edificios, Root es el esquema jerárquico, OU son divisiones, cuentas son edificios, SCP son reglas corporativas máximas, IAM es autorización individual y el rol es credencial temporal para operar en un edificio sin convertirse en su propietario. La metáfora no implica que SCP conceda permisos.

## 8. Puentes con notas ya existentes
[[AWS-Onion - LEGO Architecture Map]]
[[IAM Groups Roles STS]]
[[AWS Organizations - Evaluaciones y errores de las rondas]]
[[AWS Organizations - Capitulo semilla de libro]]

Fuente principal: transcripción aportada "[HOL] Create Organization and add Account.txt". Las distinciones adicionales proceden de la explicación arquitectónica y las evaluaciones realizadas en esta conversación, no de nuevos pasos demostrados en la clase.
