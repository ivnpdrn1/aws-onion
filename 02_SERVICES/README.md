# AWS-Onion — Catálogo LEGO de AWS Services

**Una ficha = un servicio/componente = un verbo rector y un contrato de conexiones.**

Abrir en Obsidian: [[AWS-Onion - LEGO Architecture Map]].

Las fichas de este directorio normalizan y enlazan las notas desarrolladas existentes en `01_NOTES/`, sin borrarlas. No son sustitutos de documentación técnica: son el **índice de composición** con scope, upstream, downstream, cooperación, limitaciones y disparadores de examen.

## Secuencia práctica
**Requirement → Verbo → Objeto → Scope → Interfaces → Plano → Fallo → Elección → Validación.**

Para más detalle, ver [[AWS-Onion - Verbo Rector y Service DNA]].

## Scaling / persistence LEGO blocks
- [[Amazon EFS]] — SHARE files
- [[Amazon RDS]] — PERSIST relational data
- [[Amazon DynamoDB]] — STORE NoSQL items
- [[Scaling Up vs Scaling Out - Stateful Stateless Functional Onion]] — ENLARGE vs REPLICATE and EXTERNALIZE STATE

## EC2 Auto Scaling — relación integrada
- [[EC2 Auto Scaling - Maintain Adjust Functional Onion]]
**Launch Template DEFINE → ASG MAINTAIN → CloudWatch OBSERVE / Scaling Policy DEFINE RESPONSE → ASG ADJUST → EC2 EXECUTE.** ELB DISTRIBUTE; Target Group GROUP/CHECK. Los health checks EC2 y ELB tienen alcances distintos. Consultar la nota para las modalidades manual, dynamic, predictive y scheduled y las distinciones Scaling Up/Out y stateful/stateless.

## Bootstrapping LEGO — EC2 User Data
- [[EC2 User Data - Bootstrap LEGO Service DNA]] — BOOTSTRAP: contrato, límites, upstream/downstream y decisiones de arquitectura
- [[EC2 User Data - Bootstrap Apache IMDSv2 LEGO Onion]] — guía exhaustiva Bash/Yum/Systemd/IMDSv2/HTML con alternativas Apache vs Nginx/containers, fallos, exámenes

## Reliability LEGO — HA / FT / Durability
- [[Reliability - High Availability Fault Tolerance Durability LEGO]] — ensamblaje con dominios de fallo, diferencias, trade-offs y opciones descartadas.
- [[High Availability Fault Tolerance Durability - AWS Onion LEGO]] — explicación de la dinámica y preguntas para certificación.

## Elastic Load Balancing LEGO — distribution and failure response
- [[Elastic Load Balancing - Distribute LEGO Service DNA]] — ELB DISTRIBUTE; listeners ACCEPT, Target Groups GROUP, health checks TEST, ASG MAINTAIN. Contratos de plano de datos y control, ALB vs NLB vs GWLB, decisiones y errores de examen.
- [[Elastic Load Balancing - Cuando la puerta tiene varios servidores]] — analogía didáctica y semilla de capítulo (en GitHub: `book/CHAPTER-SEED-ELB-When-One-Door-Has-Many-Servers.md`).

## ELB assessment reinforcement
- [[ELB - Assessment Reinforcement LEGO]]: six rounds; distinctions between ALB, NLB and GWLB; DNS, sessions, and fault domains.
