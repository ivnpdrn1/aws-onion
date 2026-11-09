---
type: exam-review
project: AWS-Onion
topic: AWS Organizations
updated: 2026-11-09
tags: [organizations, aws-onion-lego, evaluation, learning-gaps]
---
# AWS Organizations — Evaluaciones y errores de las rondas

## Registro de evaluación (estado guardado)
| Ronda | Aciertos | Preguntas con error | Evidencia |
|---|---:|---|---|
| 1 | 8/10 | 6 y 9 | multi-select de creación/invitación de cuenta; AccessDenied Organizations tras Switch Role |
| 2 | 8/10 | 7 y 10 | creer que OU concede permisos; creer que SCP reemplaza roles administrativos |
| 3 | **10/10** | ninguna | SCP vs IAM, Management Account, STS y confianza asumidos correctamente |

**Nivel 1 (elección múltiple): 10/10 en Ronda 3.** No se infiere dominio universal ni nivel 2 de preguntas abiertas.

## Refuerzo PRIORIDAD 1 — La cadena LIMIT ≠ AUTHORIZE
- SCP FullAWSAccess = máximo sin restricciones adicionales de esa SCP, **no** un grant.
- La OU agrupa cuentas, y políticas aplicables se heredan; **pertenecer a OU no concede permiso IAM**.
- IAM policy autoriza acciones concretas del principal según evaluación total. Deny aplicable prevalece.
- **Preguntas abiertas:** si el rol IAM solo puede S3 GetObject y SCP es FullAWSAccess, ¿puede crear bucket? Explicar paso a paso.

## Refuerzo PRIORIDAD 2 — Management Account vs Member Account
- Cuenta que crea Organizations = Management Account.
- Una sesión admin dentro de Production no convierte a Production en Management Account.
- Un AccessDenied en la consola de Organizations después de Switch Role no demuestra que AssumeRole fallara.
- **Pregunta abierta:** explicar por qué el instructor puede usar EC2/IAM pero no determinadas APIs de Organizations después de cambiar al rol Production.

## Refuerzo PRIORIDAD 3 — Crear cuenta vs invitar existente
- Cuenta nueva: correo único; Organizations la crea y puede configurar OrganizationAccountAccessRole.
- Cuenta existente: invitar y aceptar; no confundir con crear cuenta, OU ni IAM Group.
- **Pregunta abierta:** ¿qué datos necesita un administrador para Switch Role y qué autorizaciones adicionales son necesarias?

## Refuerzo PRIORIDAD 4 — Identidad temporal y confianza
- sts:AssumeRole + trust policy + permisos aplicables → sesión temporal STS.
- IAM Group no sustituye un rol; root user no es Organizations Root.
- **Pregunta abierta:** dibujar la transición entre principal origen y Role Session de Production, diferenciando política de confianza, política de permisos, SCP y STS.

## Prueba siguiente sugerida — Nivel 2
Cinco escenarios abiertos, 4 puntos por eje: VERBO/OBJETO, RESPONSABILIDAD, RELACIONES, DECISIÓN, FALLOS/LÍMITES. Exigir explicar alternativas descartadas. No afirmar que se completó sin evidencia.

## Para el libro
Narrar primero la confusión: «Una regla corporativa dice que se puede abrir una puerta; ¿quién entregó la llave?». Luego «Una tarjeta de visitante no convierte al visitante en director del edificio». Casos correctivos: SCP vs IAM, Management vs Member y crear vs invitar.
