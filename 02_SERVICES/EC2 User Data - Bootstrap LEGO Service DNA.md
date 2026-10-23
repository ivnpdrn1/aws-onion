---
tags: [aws-onion, lego, service-dna, ec2, user-data, bootstrap]
---
# EC2 User Data — BOOTSTRAP | Service DNA LEGO

**Verbo rector:** BOOTSTRAP / CONFIGURE.

**Objeto:** configuración inicial de una instancia EC2 después de boot.

**Responsabilidad exclusiva:** ejecutar instrucciones de preparación para convertir compute genérico en runtime útil de una aplicación. En Linux el agente `cloud-init` procesa normalmente user-data; los scripts suelen ejecutarse solo en el primer arranque salvo configuración especial.

**Entrada:** AMI + instancia lanzada + datos de usuario + conectividad/permisos necesarios.

**Salida:** dependencias instaladas, servicio configurado, artefactos locales y verificación operativa.

**Upstream:** [[Amazon EC2]], AMI, Launch Template, ASG; contexto de red, rol IAM cuando corresponda.

**Downstream:** OS/package manager → Apache/Nginx/runtime → configuración / contenido → systemd → puerto de servicio → Target Group / ALB / cliente.

**Contratos del LEGO:** 
- `yum install`: existe el paquete.
- `systemctl start`: proceso actual ejecutándose.
- `systemctl enable`: arranque tras reboot.
- `IMDSv2 PUT token`: token temporal para leer metadata.
- `GET placement/availability-zone`: cadena AZ propia de la EC2.
- `cat > index.html <<EOF`: genera archivo y expande `$AZ`.
- `HTTP :80`: proceso sirve tráfico solo cuando las capas de red lo permiten.

**Sustitución sin perder la esencia:** httpd → nginx cambia producto; yum → apt cambia OS; AMI con software precargado reduce bootstrap; ECS/Fargate y Lambda cambian el modelo de runtime (no copiar User Data literalmente).

**Fallo y observabilidad:** `/var/log/cloud-init-output.log` → `systemctl status httpd` → `curl localhost` → reglas/red → health de Target Group. Los checks de EC2, proceso y aplicación no son equivalentes.

**Composición del TODO:**
```text
Launch Template DEFINE
 → ASG MAINTAIN/ADJUST
 → EC2 EXECUTE
 → User Data BOOTSTRAP
 → Apache SERVE
 → Target Group CHECK
 → ALB DISTRIBUTE
 ← CloudWatch OBSERVE
```

**Decisión arquitectónica:** User Data para personalización inicial y laboratorios; AMI prehorneada/imagen de contenedor para arranques veloces y repetibles a escala; servicios administrados cuando se busca reducir gestión del OS. Elegir por boot time, fallas, costo operativo y responsabilidad de administración.

**Nota de detalle:** [[EC2 User Data - Bootstrap Apache IMDSv2 LEGO Onion]]

**Regla LEGO:** pieza = verbo + objeto + frontera + contrato + dependencia + alternativa. La suma no es solo piezas: es una cadena validable hasta la experiencia del usuario final.
