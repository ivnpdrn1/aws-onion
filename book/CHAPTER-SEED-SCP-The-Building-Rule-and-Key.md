# SCP — La regla del edificio y la llave

## Problema narrativo
Una corporación dirige numerosos edificios. Algunos son laboratorios experimentales (Development), otros guardan información muy sensible (Production). La dirección necesita establecer reglas que ningún administrador local pueda anular.

## Analogía con rigor técnico
- **Organizations** = corporación; **Root organizativo** = regla general superior.
- **OU** = divisiones; **Account** = edificios con identidades propias.
- **SCP** = techo corporativo de acciones posibles, no autorización personal.
- **IAM** = llave/permiso de una persona para abrir una puerta concreta.
- **Deny List** = por regla general se permiten actividades según permisos personales, excepto prohibiciones corporativas.
- **Allow List** = cada nivel debe contemplar la actividad, y además la persona necesita su permiso individual.

## Historia A — La prohibición que desciende
Root y OU permiten actividades ampliamente; la dirección de Finance prohíbe DeleteTable. Un administrador con llave IAM completa no puede borrar una tabla. ¿Quién decide? Deny explícito de SCP.

## Historia B — El permiso que no sube
Root autoriza EC2 y CloudWatch. Una OU solicita DynamoDB. Aunque OU e IAM lo permitan, Root no lo contempló; la operación no pasa. El lector reconstruye la **intersección entre niveles**.

## Historia C — Dos departamentos diferentes
Development necesita nuevos servicios de AWS; Production solo algunos aprobados. Si Root se limita a servicios de Production, bloquea Development. Diseñar universo superior y restricciones específicas de OU. Explicar criterios seguridad/costo/mantenimiento.

## Caja didáctica de errores reales
Resultados de evaluación: **6/10 → 10/10**.
Confusiones iniciales: SCP adjunta a rol, IAM supera la falta de Allow superior, unión de permisos entre niveles, FullAWSAccess como supuesto grant. La narración transforma cada error en una demostración práctica.

## Moraleja arquitectónica
**SCP determina el máximo posible; IAM determina la autorización del principal.** La arquitectura del gobierno no es una línea física del tráfico: es una jerarquía de decisiones.

## Glosario VERBO → OBJETO
ORGANIZE → accounts; GROUP → OU; LIMIT → SCP; AUTHORIZE → IAM; ASSUME → STS role session; EXECUTE → API.

## Experimento sugerido en sandbox
En una organización de prueba, comparar FullAWSAccess, un Deny específico y una Allow List en OU aislada. Confirmar acceso con una acción inofensiva y registrar el resultado sin modificar cuentas de producción.

## Fuentes
Diapositivas Digital Cloud Training «SCP Strategies and Inheritance», «Service Control Policy Strategies»; documentación AWS SCP evaluation. Enlazar a [[SCP Strategies and Inheritance - Architecture LEGO]].
