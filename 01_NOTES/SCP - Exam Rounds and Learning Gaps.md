---
type: exam-review
project: AWS-Onion
topic: SCP Strategies & Inheritance
date: 2026-11-09
---
# SCP — Evaluaciones y errores de las rondas

## Evolución registrada (no suposición)
| Ronda | Resultado | Estado |
|---|---:|---|
| 1 | **6/10** | por reforzar |
| 2 | **10/10** | objetivo de opción múltiple alcanzado |

**Ronda 1**: errores en preguntas 1, 3, 5 y 6.
- Q1: eligió una SCP Allow «por cada rol IAM» en lugar de Deny List con FullAWSAccess; **SCP se adjunta a Root/OU/Account, no IAM Roles**.
- Q3: confió en IAM Allow de DynamoDB pese a que Root no lo incluía en Allow List; **un descendiente no amplía el ancestro**.
- Q5: incluyó S3 aunque OU1 no lo permitía; **entre niveles la autorización es intersección**, no unión.
- Q6: marcó como correcto que FullAWSAccess da AdministratorAccess; **una SCP nunca concede permisos IAM**.

**Ronda 2**: 10/10 respuestas correctas en los diez ítems, incluyendo unión de Allows al mismo nivel, Deny explícito, OU Production vs Development, IAM vs SCP y servicios que sobreviven a la intersección.

## Cinco criterios para evaluar conocimiento real
1. **VERBO/OBJETO**: LIMIT (SCP) vs AUTHORIZE (IAM) con ejemplos.
2. **RESPONSABILIDAD**: Root, OU1, OU2 y cuenta; quién limita y quién no concede.
3. **RELACIONES**: intersección por niveles, unión de Allows en un nivel y Deny explícito.
4. **DECISIÓN**: justificar Deny List / Allow List / híbrida y alternativas descartadas.
5. **FALLOS/LÍMITES**: consecuencias de retirar FullAWSAccess sin sustituir, dependencias, testing y Management Account.

## Nivel siguiente — propuesta de examen abierto
Caso: Organización de cuatro cuentas; OU Dev flexible y OU Prod regulada; rol con IAM AdministratorAccess; Root tiene universo S3/EC2/CloudWatch; Prod necesita servicio nuevo. Pedir explicación completa, corregir la arquitectura y decidir si ampliar Root o separar diseño. Puntuar cinco ejes × 4 puntos; exigir 20/20 antes de declarar dominio de diseño.

## Conceptos que deben sobrevivir como memoria permanente
- **SCP LIMIT, IAM AUTHORIZE**.
- **Deny explícito gana** a Allow IAM.
- **Allow debe existir en CADA nivel**, no en CADA SCP individual.
- **Varias SCP en un nivel aportan Allows**, sujeto a Deny aplicable.
- **Menos permisos en Root → ninguna OU descendiente puede recuperar acciones excluidas**.
- **FullAWSAccess no es AdministratorAccess**.

[[SCP Strategies and Inheritance - Architecture LEGO]]
