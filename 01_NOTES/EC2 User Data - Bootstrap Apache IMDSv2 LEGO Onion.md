---
tags: [aws-onion, ec2, user-data, bootstrap, linux, imdsv2, apache, lego, saa]
aliases: ["EC2 User Data LEGO", "Bootstrapping Web Server"]
---
# EC2 User Data — Bootstrapping, Apache e IMDSv2 | LEGO AWS-ONION

> **Idea rectora:** User Data transforma una instancia EC2 genérica en un servidor con una responsabilidad específica, de manera repetible. **No es Apache el concepto fundamental: es la composición PREPARAR → INSTALAR → CONFIGURAR → INICIAR → DESCUBRIR → GENERAR → SERVIR → VALIDAR.**

## 1. Vista unitaria: LEGO → TODO

```text
Launch Template (DEFINE configuration) → EC2 (PROVISION compute)
    → cloud-init / User Data (BOOTSTRAP)
    → OS + package manager (PREPARE)
    → Apache httpd (SERVE HTTP)
    → IMDSv2 (DISCOVER instance context: AZ)
    → heredoc + shell expansion (GENERATE index.html)
    → systemd (RUN + ENABLE)
    → HTTP client / Target Group / ALB (CONSUME + DISTRIBUTE)
```

Cada bloque tiene una frontera contractual: **entrada, acción, salida, dependencia y fallo**. La eficiencia del TODO depende de automatización reproducible, responsabilidad única, conectividad, observabilidad y capacidad de reemplazo.

## 2. Script reconstruido y comentado

```bash
#!/bin/bash
# Selecciona intérprete Bash (shebang).
yum update -y
# Actualiza paquetes instalados en Amazon Linux compatible; puede alargar bootstrap.
yum install -y httpd
# Instala el paquete Apache HTTP Server ("httpd").
systemctl start httpd
# Arranca el servicio AHORA.
systemctl enable httpd
# Permite que arranque automáticamente tras reiniciar.

# IMDSv2: solicitar token por HTTP PUT a una IP link-local reservada para metadata.
TOKEN=$(curl -fsS -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
# TOKEN captura stdout. TTL 21600 segundos = 6 horas; no es credencial IAM.

# Consultar availability zone pasando el token por cabecera.
AZ=$(curl -fsS -H "X-aws-ec2-metadata-token: $TOKEN" \
  "http://169.254.169.254/latest/meta-data/placement/availability-zone")
# AZ contiene un valor como us-east-1a (ejemplo); no es una región ni una IP.

cat > /var/www/html/index.html <<EOF
<html>
<head>
  <title>Instance Availability Zone</title>
  <style>
    body {
      background-color: #6495ED;
      color: white;
      font-size: 36px;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      font-family: Arial, sans-serif;
    }
  </style>
</head>
<body>
  <div>This instance is located in Availability Zone: $AZ</div>
</body>
</html>
EOF
# heredoc: <<EOF abre entrada multilínea; EOF cierra.
# > sobrescribe el archivo; Bash expande $AZ porque EOF no está entrecomillado.
# Ruta /var/www/html/index.html = document root habitual de Apache en Amazon Linux.
# systemctl enable httpd ya cubre persistencia; chkconfig httpd on del original es redundante.
```

**Correspondencia con capturas:** original usa backticks (`TOKEN=\`curl ...\``): preferir `$(...)` por legibilidad y anidamiento. Original usa `chkconfig httpd on` además de `systemctl enable httpd`; en systemd moderno no es necesario repetir. Esta versión ilustra la semántica del original y agrega `-fsS` para fallar de forma más explícita, pero una instalación productiva exige manejo de errores adicional.

## 3. Línea → verbo → objeto → porqué → reemplazo

| Línea / sintaxis | Verbo | Objeto | Justificación | Alternativa / frontera |
|---|---|---|---|---|
| `#!/bin/bash` | INTERPRETAR | Script | El kernel sabe qué intérprete ejecutar al iniciar el archivo ejecutable | `/bin/sh`, Python con sintaxis apropiada |
| `# ...` | DOCUMENTAR | Código | Aclara intenciones; no se ejecuta | — |
| `yum update -y` | ACTUALIZAR | Paquetes del OS | Reduce obsolescencia; no reemplaza política de parchado | `dnf`, `apt`; versiones fijadas/AMI prehorneada |
| `yum install -y httpd` | INSTALAR | Servidor web | Proporciona runtime HTTP | Nginx, Node.js, contenedor |
| `systemctl start httpd` | ARRANCAR | Servicio | Produce estado presente activo | `systemctl start nginx` |
| `systemctl enable httpd` | HABILITAR | Servicio | Produce comportamiento futuro tras reboot | Unidad systemd u orquestador |
| `curl -X PUT .../api/token` | SOLICITAR | Token IMDSv2 | Establece sesión de consulta de metadata | SDK para ciertos metadatos; no confundir con autorización IAM |
| `-H ...ttl-seconds: 21600` | CONFIGURAR | Token | Expiración 6 horas | TTL menor para consulta breve |
| `TOKEN=$(...)` | CAPTURAR | Salida de comando | Reutiliza resultado dinámico | variables de entorno/configuración externa |
| `curl -H ...$TOKEN .../placement/availability-zone` | DESCUBRIR | AZ de esa EC2 | Evita hardcode y permite instancias en distintas AZ | Config explícita donde convenga |
| `cat > .../index.html <<EOF` | GENERAR | Archivo HTML | Escribe contenido a servir; `>` reemplaza, `<<EOF` delimita | Template engine, artefacto compilado, S3 |
| `<title>` | IDENTIFICAR | Página | Título de pestaña | Framework frontend |
| CSS `background/color/font` | PRESENTAR | Interfaz | Aspecto visual; NO arquitectura AWS | hoja CSS separada |
| CSS `display:flex`, `justify-content`, `align-items` | ALINEAR | Contenido | Centra en ejes principal y transversal | Grid |
| `height:100vh`, `margin:0` | DIMENSIONAR | Viewport | Centrado a toda altura sin margen | CSS moderno |
| `<div>...$AZ</div>` | MOSTRAR | Dato de entorno | Acopla metadata al contenido servido | Endpoint dinámico o template |
| `EOF` | CERRAR | Heredoc | Finaliza el bloque de texto | Otra marca única |
| `chkconfig httpd on` | HABILITAR | Servicio | Legacy/redundante tras `systemctl enable` | Omitir en systemd |

### Diferencias decisivas
- **`start` ≠ `enable`:** encender ahora vs arrancar en siguientes boots.
- **Paquete ≠ servicio:** `yum install` instala bits; `systemctl` gestiona el proceso.
- **User Data ≠ AMI:** AMI aporta imagen base; User Data personaliza al iniciar. La ejecución de scripts User Data vía cloud-init normalmente ocurre en el primer arranque, no en todos los reinicios, salvo configuración expresa.
- **IMDSv2 token ≠ IAM access key:** token habilita solicitudes de metadata local, no es un principal IAM ni concede permisos de AWS API.
- **AZ ≠ Region:** AZ es ubicación concreta dentro de una región; su nombre puede mapearse distinto entre cuentas AWS.
- **Metadata IP ≠ IP de la instancia:** 169.254.169.254 es destino link-local de IMDS, no IP pública ni privada de la EC2.
- **`>` ≠ `>>`:** reemplazar vs agregar contenido.
- **`<<EOF` ≠ `<<'EOF'`:** con delimitador sin comillas se expande `$AZ`; con delimitador citado se mantiene literalmente.
- **Path ≠ ejecutable siempre:** `/bin/bash` es archivo ejecutable; `/var/www/html/index.html` es archivo de contenido; el significado depende del tipo de objeto, no solo del slash final.

## 4. Onion de ejecución, dependencia y acceso

```text
CONSTRUCCIÓN: Launch Template/AMI → boot EC2 → cloud-init/User Data
                                ↓
APLICACIÓN:       OS → Package manager → httpd → index.html
                                ↑               ↑
CONTEXTO:                   IMDSv2 TOKEN → AZ → shell variable
                                ↓
TRÁFICO: DNS (si existe) → ALB (si existe) → Target Group → EC2 ENI → httpd:80
           └ Internet/IGW/routes/NACL/SG deben permitir el flujo correspondiente
```

**Nota de precisión:** el diagrama de capas de red es un modelo de diagnóstico, no un orden físico obligatorio de aparatos; para un ALB público con targets privados la ruta real difiere de una EC2 pública accedida directamente.

El script **no configura** VPC, subnets, rutas, IGW, SG, NACL, IP pública, DNS, balanceador, registro de targets ni HTTPS. EC2 con Apache activo puede seguir inaccesible desde Internet. Una instancia privada puede servir mediante ALB sin IP pública.

## 5. Alternativas: preservar responsabilidad, sustituir implementación

| Responsabilidad | Ejemplo original | Alternativa |
|---|---|---|
| Compute | EC2 | ECS Fargate, Lambda (con modelo operativo distinto) |
| Imagen del runtime | Amazon Linux | Ubuntu, imagen Docker |
| Paquetes | yum | dnf, apt, imagen preconstruida |
| HTTP server | Apache/httpd | Nginx, Node.js |
| Inicio y persistencia | systemctl | Entrypoint de contenedor/orquestación |
| Personalización inicial | EC2 User Data | AMI baking, container image, IaC, configuración gestionada |
| Información de entorno | IMDSv2 | variables de entorno/config inyectada según plataforma |
| Contenido | index.html | aplicación SSR, backend/API, contenido en S3 |
| Exposición | SG + HTTP directo | ALB + Target Group + TLS/ACM |

**No son intercambios 1:1:** Fargate no utiliza EC2 User Data del mismo modo; Lambda no funciona como Apache permanente; S3 estático no ejecuta Bash/Metadata; ECS usa tareas y task metadata, no el mismo modelo de acceso de instancia.

## 6. LEGO de Auto Scaling

```text
REQUIREMENT: mantener N servidores sanos ante variación de carga
Launch Template DEFINE (AMI + User Data + SG)
   → ASG MAINTAIN / ADJUST (desired capacity y reemplazo)
   → EC2 BOOT (ejecuta User Data en lanzamiento)
   → Apache SERVE (contenido con AZ propia)
   → Target Group REGISTER + CHECK
   → ALB DISTRIBUTE solicitudes a targets sanos
   ← CloudWatch OBSERVE → policy decide escala → ASG actúa
```

Un mismo script permite instancias idénticas en responsabilidad, aunque cada AZ varíe. Si se ven AZ diferentes al refrescar, eso **puede** ilustrar distribución, pero no la garantiza por caché, afinidad, balanceo y comportamiento del cliente. Un ejemplo de contenido estático es stateless; cualquier sesión o dato persistente de aplicación real debe diseñarse explícitamente.

## 7. Calidad operativa y eficiencia

1. **Bootstrap reproducible e idempotente:** evitar instalación repetida innecesaria, fijar versiones cuando corresponda, gestionar fallo de descargas, logs y reintentos.
2. **Orden de dependencias:** si falla metadata, la página puede quedar con AZ vacía; comprobar token, respuesta y servicio.
3. **Evitar credenciales en User Data:** metadata de IAM role (si existe) usa credenciales temporales; minimizar privilegios; IMDSv2 requerido y hop limit adecuado.
4. **Tiempo de arranque:** actualizar todos los paquetes con cada escalado puede retrasar el registro en el Target Group. AMI prehorneada o contenedor reduce latencia.
5. **Exposición y cifrado:** controlar puertos, TLS, SG mínimo; no asumir que EC2 pública es requisito del ALB.
6. **Observabilidad:** `/var/log/cloud-init-output.log`, `systemctl status httpd`, `curl localhost`, logs Apache, EC2 status checks y ALB target health son observaciones distintas.
7. **Orden preferible:** generar/configurar página y dependencias, luego arrancar servicio y probarlo; script original arranca Apache antes de escribir HTML, con posible ventana de página predeterminada.
8. **Verificación:** el estado `active` de httpd no comprueba reachability ni salud del ALB: pruebas en cada frontera.

## 8. Diagnóstico por piezas

| Síntoma | Pieza y prueba | Diagnóstico siguiente |
|---|---|---|
| EC2 corre pero página no responde | SG, routing, IP/ALB, NACL | Llegar al puerto 80 |
| `curl localhost` falla | `systemctl status httpd`; paquetes | ¿Servidor instalado/activo? |
| Apache responde página genérica | `cat /var/www/html/index.html` | User Data/heredoc |
| AZ en blanco | `TOKEN`, IMDSv2 habilitado, `curl` errores | metadata o shell expansion |
| Target unhealthy | Health-check path/port, SG de targets, logs | Contrato ALB → target |
| Instancia nueva tarda | cloud-init logs, repos de paquetes, tiempos de dependencia | AMI precocinada/bootstrap mínimo |
| Funciona antes de reiniciar, después no | `systemctl is-enabled httpd` | Persistencia del servicio |

## 9. Preguntas de certificación SAA/DVA
- ¿Por qué usar **Launch Template + User Data** para una ASG? Para repetir inicialización sin configurar máquinas a mano.
- ¿Por qué **AMI preconstruida** puede ser mejor en una flota grande? Menos dependencias de red y menor bootstrap time, con costo de mantener imágenes actualizadas.
- ¿Qué detecta **ALB health check** vs **EC2 status check**? Respuesta de aplicación/config del target vs salud de instancia/host subyacente.
- ¿Qué escala: ALB o ASG? ALB **distribuye**, ASG **mantiene y ajusta capacidad**.
- ¿Qué protege IMDSv2? Acceso a metadata mediante solicitud de token; no reemplaza SG, IAM o controles de aplicación.
- ¿Cómo evitar estado perdido al reemplazar EC2? Externalizar datos/sesiones a servicios persistentes adecuados.
- ¿Por qué no expone Internet el propio User Data? El plano de red y seguridad es independiente del runtime de aplicación.

## 10. Relaciones en el Vault

- [[AWS-Onion - LEGO Architecture Map]]
- [[AWS-Onion - Verbo Rector y Service DNA]]
- [[EC2 Auto Scaling - Maintain Adjust Functional Onion]]
- [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]]
- [[EC2 User Data - Bootstrap LEGO Service DNA]]

**Frase de memoria:** *AMI define de qué nace; User Data qué hace al nacer; systemd qué corre y reinicia; IMDS qué contexto descubre; Apache qué sirve; SG/rutas quién puede conectarse; ALB a cuál target enviar; ASG cuántos mantener.*