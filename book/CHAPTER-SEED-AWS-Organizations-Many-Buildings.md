# AWS Organizations — La empresa con varios edificios

## Necesidad antes de servicio
Una empresa comenzó en un solo edificio: desarrollo y producción mezclados. Crece y necesita separar riesgos y administración, pero mantener reglas corporativas.

## Analogía LEGO
- **Organizations ORGANIZE:** la corporación crea una estructura común.
- **Management Account MANAGE:** dirección que administra la estructura.
- **Root organizativo ANCHOR:** nodo superior del organigrama, NO el dueño/root user.
- **OU GROUP:** divisiones que reúnen edificios/cuentas.
- **Member Account ISOLATE:** cada edificio tiene recursos e identidades propias.
- **SCP LIMIT:** límite corporativo a lo permitido en las cuentas miembro; NO entrega llaves.
- **IAM AUTHORIZE:** autoriza individualmente una acción.
- **Role + STS ASSUME:** credencial temporal para trabajar en otro edificio.
- **Switch Role:** interfaz para tomar ese contexto temporal.

## Conflicto pedagógico — la política que parece conceder todo
«FullAWSAccess permite todo, entonces mi rol ya es administrador». Error: SCP no concede. Necesito una autorización IAM. El lector debe recorrer ambas capas sin confundir «techo de permisos» con «permiso otorgado».

## Segundo conflicto — ¿Quién administra a quién?
El administrador entra en Production mediante OrganizationAccountAccessRole, puede operar EC2 y ve AccessDenied al intentar una tarea propia de Organizations. No significa que falló la sesión: está administrando una **cuenta miembro**, no la **organización**.

## Escena de laboratorio reproducible
Crear organización; examinar Root y Management; habilitar SCP y FullAWSAccess; crear cuenta Production con correo único; localizar el ID y OrganizationAccountAccessRole; cambiar rol y comprobar permisos; volver. **No usar cuentas de producción reales sin control de costos y seguridad.** La clase original muestra este recorrido; las discusiones de evaluación añaden interpretaciones de arquitectura.

## Cinco preguntas del lector
1. ¿Qué frontera necesito: OU, IAM Group o cuenta?
2. ¿Qué diferencia una SCP de una IAM Policy?
3. ¿Quién puede asumir el rol y bajo qué confianza?
4. ¿Qué cambia en STS, y qué NO cambia en Organizations?
5. ¿Cómo se demuestra que un Deny corporativo tiene efecto sin otorgar permisos nuevos?

## Material empírico del método
Rondas sobre este capítulo: **8/10 → 8/10 → 10/10**. Errores de aprendizaje agrupados en: crear/invitar cuenta, alcance de Management Account, SCP FullAWSAccess vs permisos IAM, y rol asumible vs reglas organizativas.

Estructura editorial: PROBLEMA → VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → ERROR INTUITIVO → DECISIÓN → PRUEBA.
