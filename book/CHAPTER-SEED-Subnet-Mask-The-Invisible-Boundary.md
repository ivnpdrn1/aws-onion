---
type: book-chapter-seed
project: AWS-ONION-LEGO
chapter: "La frontera invisible de la red"
status: draft
updated: 2026-11-13
tags: [aws-onion, book, networking, cidr, subnet, learning-story]
---
# Capítulo semilla — La frontera invisible de la red

## Apertura narrativa: una dirección no cuenta toda la historia

Dos vecinos escriben una dirección en un papel: `192.168.0.5`. Uno pregunta: «¿Cuál es su barrio?». La dirección, por sí sola, no lo dice. Hace falta conocer el **límite del barrio**. En IPv4 ese límite se describe con una máscara de subred o prefijo CIDR.

La dirección cuenta *dónde está* una interfaz. La máscara dice *qué parte de la dirección identifica la red*. La tabla de rutas responde *cómo llegar* cuando el destino queda fuera del ámbito local. Son preguntas diferentes.

## Primera pieza — SEPARATE

`192.168.0.5` bajo `255.255.255.0` (=/24) pertenece a `192.168.0.0/24`. De los 32 bits, 24 identifican el prefijo y 8 forman la parte de host: `2^8 = 256` direcciones en el bloque. La máscara no es un portero y no decide quién entra.

## El giro — dividir el barrio

Una decisión de arquitectura exige separar la red en dos segmentos del mismo tamaño. Se toma un bit adicional para el prefijo: `/24` se convierte en dos `/25`.

```text
192.168.0.0/24 (256 IP)
  ├── 192.168.0.0/25 (128 IP)
  └── 192.168.0.128/25 (128 IP)
```

Una máscara más larga resulta en **cada bloque más pequeño**, no más grande.

## El encuentro con AWS

La organización crea una VPC `10.0.0.0/16` y dos subnets en AZ distintas: `10.0.1.0/24` y `10.0.2.0/24`. Ahora cada segmento debe tener su propia capacidad y su posición en la arquitectura. AWS reserva cinco direcciones IPv4 **por subnet**: un /24 ofrece 251 asignables; un /25, 123.

La pieza **Subnet SEGMENT** necesita un bloque de direcciones; **Route Table ROUTE** decide el camino; **Security Group / NACL FILTER** aplican reglas; **ENI CONNECT** entrega una interfaz y dirección al recurso. Ninguna pieza sustituye a otra.

## El conflicto pedagógico

Se pide colocar 180 interfaces con IPv4 privada en una subnet. El arquitecto elige /25 «porque 25 es mayor que 24». Descubre que /25 solo ofrece 123 IPv4 asignables en AWS. Corrige la decisión a /24 con 251 asignables y contempla margen de crecimiento.

## Preguntas para el lector

- ¿Qué cambia exactamente cuando se aumenta el prefijo de /24 a /25?
- ¿Por qué tener una dirección dentro de la VPC no garantiza salida a Internet?
- ¿Qué componente determina la red, cuál decide el camino y cuál permite el tráfico?
- ¿Dónde termina la analogía del barrio? Las reglas reales de AWS, routing y seguridad no son una única barrera.

## Cierre

> **No basta con conocer la dirección. Hay que conocer el límite de la red y las reglas para viajar entre límites.**

### Receta editorial AWS-ONION-LEGO
NECESIDAD → VERBO (**SEPARATE**) → OBJETO (bits IPv4) → RESPONSABILIDAD (prefijo) → RELACIONES (VPC/Subnet/Route Table) → DECISIÓN (/24 vs /25) → FALLO (insuficiente capacidad) → EXAM CRAM.

**Base técnica:** [[Subnet Mask CIDR Network ID Host ID - AWS ONION LEGO]].  
**Continuidad:** [[Subnet Mask and CIDR - Define Network Boundary LEGO]], [[Subnet]], [[Route Table]].
