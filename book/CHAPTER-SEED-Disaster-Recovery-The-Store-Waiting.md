# Semilla de capítulo — La tienda que espera apagada

## Pregunta central
¿Qué pasa cuando no falla una instancia, ni una AZ, sino toda una región?

**Escena**: una tienda en Virginia trabaja todos los días. En otra ciudad hay planos, copias del inventario y una organización lista para contratar personal, pero ninguna persona trabajando. Esa es la intuición de un ASG secundario con capacidad 0.

## Narración y límites
1. **DISTINGUIR DOMINIOS**: Multi-AZ distribuye riesgo dentro de región; Multi-Region prepara recuperación geográfica.
2. **PRESERVE**: snapshots copiados entre regiones preservan ciertos datos, no toda la aplicación.
3. **DEFINE**: AMI y Launch Template preparados especifican cómo lanzar EC2; todas las dependencias deben ser utilizables en región B.
4. **MAINTAIN**: ASG en cero ahorra capacidad EC2 pero aumenta RTO.
5. **RESTORE**: levantar datos y servicios, lanzar EC2, validar Target Group y ALB.
6. **RESOLVE / FAILOVER**: enviar clientes a B solo cuando la recuperación esté lista.
7. **MEASURE**: RTO tiempo de recuperación; RPO pérdida admisible de datos.

## Giro didáctico
Un estudiante concluye «snapshots + ASG 0 = pilot light siempre». Otro pregunta: «¿qué componentes críticos siguen realmente encendidos?». Sin esa información, puede tratarse de backup/restore parcialmente preconfigurado. La arquitectura se decide por lo que sobrevive, no por la etiqueta.

## Descartes
- Multi-AZ no cubre caída regional completa.
- ASG no es multirregional por sí mismo.
- CloudWatch no copia datos ni enciende automáticamente ASG B sin orquestación.
- Route 53 no restaura datos o EC2.
- Snapshot EBS ≠ base de datos transaccional restaurada y consistente.

## Actividad para el lector
Dibujar dos regiones y clasificar los recursos en datos, control, tráfico y dependencias. Comparar RTO/RPO y costo de backup & restore, pilot light, warm standby y active-active. Proponer un ensayo trimestral de recuperación y evidencia que debe conservarse.

**Basado en la diapositiva** «Architecture Patterns - Auto Scaling and ELB», con aclaraciones arquitectónicas explícitamente adicionales.
