# AWS Control Tower — El arquitecto de los edificios

## Problema inicial
Una corporación comienza con un edificio y crece a decenas de edificios. Organizations crea el organigrama y las divisiones, pero siguen faltando reglas homogéneas de construcción, registro, supervisión y apertura de edificios nuevos.

## El protagonista LEGO
**Control Tower ORCHESTRATE**: coordina la preparación de la ciudad corporativa (Landing Zone) sin reemplazar al responsable de organizarla (Organizations). La historia comienza siempre con la necesidad, no con el nombre de AWS.

## Personajes / responsabilidades
- Organizations ORGANIZE: organigrama de edificios (cuentas), Root y departamentos (OU).
- SCP LIMIT: reglas corporativas; no son llaves personales.
- IAM AUTHORIZE e Identity Center ACCESS: quién puede entrar/actuar.
- Control Tower STANDARDIZE: normas de apertura y mantenimiento.
- Account Factory PROVISION: nueva sucursal con preparación repetible.
- Management MANAGE: oficina corporativa.
- Log Archive PRESERVE: archivo protegido de evidencias.
- Audit INSPECT: inspectores autorizados.
- CloudTrail RECORD: bitácora de eventos.
- Config EVALUATE: comprobar el estado de las instalaciones.
- Preventive: impedir acción prohibida; Detective: reportar incumplimiento; Proactive: validar antes del aprovisionamiento.

## Tres escenas ilustrativas
1. Una sucursal altera una configuración: CloudTrail registra actividad, Config puede señalar incumplimiento, Audit revisa, Log Archive conserva registros. Las cuatro piezas no son redundantes.
2. Una empresa necesita 50 sucursales: Account Factory estandariza apertura; Organizations mantiene jerarquía y límites.
3. Una regla corporativa bloquea una API con SCP; la supervisión de Config puede además alertar de otro tipo de desviación. Bloquear ≠ detectar ≠ corregir.

## Elección arquitectónica
Con pocas cuentas y automatización propia, Organizations + herramientas de IaC pueden ser suficientes. Con crecimiento y baseline común, Control Tower orquesta landing zone, cuentas compartidas y controles. Examinar costos, recursos gestionados, compatibilidad y responsabilidad operacional.

## Advertencia sobre analogías
No inferir que Management Account tiene acceso indiscriminado a toda identidad o recurso. Audit no es almacén principal de logs. Security OU de la diapositiva ejemplifica una configuración, no necesariamente una jerarquía obligatoria en toda versión.

## Prueba pedagógica
Quiz sobre Control Tower: **Ronda 1, 10/10**. Se exige después explicar escenarios abiertos y los servicios descartados. Guion: NECESIDAD → VERBO → OBJETO → RESPONSABILIDAD → RELACIONES → DECISIÓN → LÍMITES → EXAMEN.

Conectar el capítulo con [[AWS Organizations - La empresa con varios edificios]] y [[SCP - La regla del edificio y la llave]].
