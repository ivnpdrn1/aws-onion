---
type: architecture-service-dna
project: AWS-Onion
topic: SCP Strategies and Inheritance
date: 2026-11-09
tags: [aws-onion-lego, organizations, scp, allow-list, deny-list, inheritance, iam]
---
# SCP Strategies and Inheritance — Gobernar sin conceder

## 1. La necesidad ANTES de la herramienta
Una empresa tiene varias cuentas. IAM autoriza acciones en cada cuenta, pero el negocio necesita imponer **límites centrales** sobre lo que ninguna identidad de sus cuentas miembro debe poder hacer. El problema no es crear otra identidad ni filtrar tráfico de red: es **gobernar el alcance máximo de los permisos**.

**Service DNA:** SCP = LIMIT / máximo organizativo; IAM = AUTHORIZE / permisos del principal. Una SCP **no concede** permisos IAM.

## 2. ONION de gobierno (no es un recorrido de paquetes)
```text
              AWS ORGANIZATIONS
                    |
            ROOT organizativo
                    |    [SCPs Root]
                    v
                   OU1
                    |    [SCPs OU1]
                    v
                   OU2
                    |    [SCPs OU2]
                    v
              MEMBER ACCOUNT
                    |    [SCPs Account]
                    v
              IAM PRINCIPAL
                    |    [IAM policies, otras restricciones]
                    v
              AWS API ACTION
```
Cada nivel plantea una pregunta distinta: **¿qué se permite como máximo?** Root/OU/Account limitan. IAM y otros controles deciden **¿esta identidad puede hacer la operación concreta?**

## 3. Dos estrategias: qué queremos permitir por defecto
| Estrategia | Configuración característica | Ventaja | Costo/riesgo | Cuándo |
|---|---|---|---|---|
| **Deny List** | FullAWSAccess en todos los niveles + SCP Deny específicas | Flexibilidad, soporta nuevos servicios sin ampliar Allow Lists | Vigilar nuevas capacidades y excepciones | Dev, investigación, controles guardrail |
| **Allow List** | Sustituir FullAWSAccess por Allow explícitos en los niveles donde se requiera el límite | Gobernanza restrictiva, aprobaciones explícitas | Mantenimiento, dependencia de acciones necesarias | producción crítica, entornos regulados |
| **Híbrida** | Root permite un universo amplio; OU de Producción define subset y Deny específicos | distintos controles para Dev/Prod | pruebas y disciplina de políticas | empresas multicuentas |

## 4. El mecanismo real: unión EN un nivel, intersección ENTRE niveles
En cada nivel, varias SCPs pueden aportar Allow, pero cualquier **Deny explícito aplicable** prevalece.
Entre niveles Root → OU1 → OU2 → Account se requiere **algún Allow que cubra la acción en cada nivel**; un nivel inferior no puede ampliar el máximo de un ancestro.

```text
PERMISOS SCP EFECTIVOS
 = (Allows Root) ∩ (Allows OU1) ∩
   (Allows OU2) ∩ (Allows Account)
   MENOS cualquier Deny explícito aplicable

PERMISO REAL
 = autorización IAM y otras políticas aplicables
   ∩ límites SCP efectivos
```

Este es un MODELO MENTAL simplificado de evaluación de acciones. No implica que cada política SCP individual deba contener un Allow: **basta algún Allow por nivel** y ausencia de Deny aplicable.

## 5. Ejemplo A: Deny List
```text
ROOT      FullAWSAccess
   |
OU1       FullAWSAccess + Deny dynamodb:*
   |
OU2       FullAWSAccess
   |
ACCOUNT   FullAWSAccess
   |
ROLE IAM  Allow dynamodb:PutItem
   |
RESULTADO DENIED — Deny de OU1 se hereda
```
El IAM Role puede seguir teniendo Allow; el gobierno central bloquea. No hay que borrar ni editar el IAM Role para imponer el límite.

## 6. Ejemplo B: Allow List
```text
ROOT      Allow ec2:* + cloudwatch:*
   |
OU1       Allow ec2:* + cloudwatch:*
   |
OU2       Allow dynamodb:* + ec2:*
   |
ACCOUNT   FullAWSAccess
   |
ROLE IAM  AdministratorAccess
   |
RESULTADO DynamoDB DENIED (Root y OU1 no lo permiten)
           EC2 podría estar permitido, sujeto a otros controles
```
Añadir un Allow DynamoDB en OU2 no amplía Root. Si Root permite solo EC2 y CloudWatch, conservar FullAWSAccess en OU2 tampoco restituye otros servicios.

## 7. Ejemplo C: arquitectura por sensibilidad
```text
ROOT FullAWSAccess
  |
  +-- OU Development: FullAWSAccess + Deny específicos
  |       -> posibilidad de probar nuevos servicios si IAM autoriza
  |
  +-- OU Production: Allow EC2/S3/CloudWatch y Deny operaciones críticas
          -> solo subconjunto aprobado, siempre sujeto a IAM
```
En un diseño real, conservar Allow en todos los niveles (cuentas incluidas) y evaluar dependencias de servicios. Antes de restringir Root, verificar que el universo permitido incluya necesidades de **todas** las OU. Adoptar cambios paulatinamente en cuentas de prueba.

## 8. Decisiones arquitectónicas y alternativas descartadas
- **Quiero restringir una operación de todas las cuentas Finance** → SCP Deny en OU Finance. No IAM Group: agrupa usuarios, no cuentas.
- **Quiero permitir solo servicios aprobados en PROD** → Allow List efectiva en su ruta. No basta un Allow en OU2 si Root lo excluye.
- **Tengo FullAWSAccess y mi rol carece de IAM Allow** → acción DENIED. No confundir máximo con autorización.
- **Quiero permitir Dev más servicios que Prod** → no recortar Root a solamente el subconjunto de Prod.
- **Quiero limitar DeleteTable pero conservar lectura DynamoDB** → Deny dynamodb:DeleteTable en lugar de dynamodb:*.

## 9. Límites que evita simplificar en exceso
- SCP se aplica a cuentas **miembro** en su ámbito; no restringe principales de la Management Account.
- SCP es techo de permisos, no herramienta para dar acceso a recursos; IAM (y otras políticas pertinentes) siguen siendo necesarias.
- Root de Organizations ≠ root user de AWS Account.
- Para Deny List también se requiere que los Allow permanezcan en los niveles; quitar FullAWSAccess de un nivel sin sustituto puede bloquear todo debajo.
- Se debe probar impacto en servicios, roles de operación y automatizaciones antes de desplegar restricciones amplias.

## 10. Referencias del curso y verificación
Las dos diapositivas proporcionadas del curso Digital Cloud Training: «SCP Strategies and Inheritance» y «Service Control Policy Strategies» muestran Deny List / Allow List, FullAWSAccess y ejemplos DynamoDB/EC2/CloudWatch. Los criterios de evaluación y matices de múltiples SCP por nivel se contrastaron con AWS:
https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_evaluation.html
https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture/organizations-security.html

## 11. Conexiones en el Vault
[[AWS Organizations - Organize Govern Assume LEGO Service DNA]]
[[SCP - Evaluaciones y errores de las rondas]]
[[SCP - La regla del edificio y la llave]]
