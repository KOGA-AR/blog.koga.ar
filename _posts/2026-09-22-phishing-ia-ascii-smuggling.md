---
layout: post
title: "Phishing con IA y ASCII Smuggling: facturas falsas que ni el filtro ni el ojo ven"
date: 2026-09-22
categories: [news, capacitaciones]
tags: [news, phishing, inteligencia-artificial, ascii-smuggling, ingenieria-social, microsoft, capacitaciones]
image:
  path: /assets/phishing-ia-ascii-smuggling.webp
  alt: "Phishing con IA y ASCII Smuggling"
author: c4cker
---

Microsoft identificó una campaña de phishing que mandó más de un millón de correos personalizados
a departamentos de cuentas por pagar, usando IA generativa para fabricar facturas falsas
indistinguibles de las reales. En paralelo, investigadores documentaron millones de correos de
phishing que esconden palabras clave dentro del texto usando caracteres Unicode invisibles — un
truco que rompe los filtros basados en detección de palabras sin que la víctima note nada raro en
pantalla.

Son dos técnicas distintas, pero apuntan al mismo objetivo: que ni el filtro automático ni el
usuario del otro lado detecten el engaño.

---

## Cómo funciona el ataque con IA

La campaña que rastreó Microsoft combina varias capas de suplantación en un mismo correo:

- **Impersonación ejecutiva**: el nombre de un directivo real de la empresa aparece en el nombre
  del remitente, en el campo "responder a" y en la firma del correo.
- **Facturas generadas por IA**: plantillas de ServiceNow con branding, logos, números de factura,
  fechas y conceptos que parecen legítimos — incluyendo la sección "Facturado a" personalizada con
  el nombre real de la organización destinataria.
- **Hilo de conversación fabricado**: el correo no llega solo. Viene acompañado de intercambios
  simulados que le dan contexto y credibilidad a la factura, bajando la guardia de quien la recibe.
- **Cuentas de destino rotativas**: las instrucciones de pago apuntan a cuentas en distintas
  entidades financieras según el objetivo, dificultando el rastreo del dinero.

El objetivo es desviar transferencias de aproximadamente 50.000 dólares por víctima. La IA no
inventa la estafa — el fraude de facturas existe hace décadas — pero elimina el trabajo manual de
armar cada pieza, permitiendo escalar el engaño a un millón de correos sin perder personalización.

---

## ASCII smuggling: lo que ni el filtro ni vos ven

La segunda técnica documentada resuelve un problema distinto para el atacante: cómo esconder
palabras que dispararían una alerta.

El método inserta caracteres Unicode invisibles — "tag characters" que no tienen representación
visual — intercalados dentro de palabras clave como "funding". Para el ojo humano, la palabra se
sigue leyendo perfecta. Para un filtro que busca coincidencias exactas de texto, la secuencia de
bytes ya no coincide con nada conocido: la palabra quedó rota por caracteres que no existen en la
pantalla pero sí en el código subyacente.

Los sistemas de detección basados en coincidencia de palabras o expresiones regulares no la
detectan porque la secuencia de caracteres contiguos se interrumpe. Los clasificadores basados en
machine learning tampoco, salvo que analicen una captura visual del mensaje en vez de procesar el
texto crudo. Microsoft reportó millones de correos usando esta técnica — no es un caso aislado, es
un método ya adoptado a escala por atacantes.

---

## Por qué esto importa más que el phishing de siempre

Ninguna de las dos técnicas depende de malware, exploits ni vulnerabilidades de software. Dependen
de que una persona lea un correo, confíe en lo que ve y actúe — transferir dinero, hacer clic,
responder con datos. La IA generativa bajó el costo de producir el engaño perfecto; el ASCII
smuggling bajó el riesgo de que ese engaño quede atrapado en un filtro antes de llegar a destino.

La superficie de ataque no cambió. Cambió qué tan barato y efectivo es explotarla.

---

## Qué hacer con esto

- Verificar cambios de cuenta bancaria o instrucciones de pago por un canal distinto al correo —
  una llamada telefónica al número ya conocido del proveedor, no al que figura en el mensaje.
- Desconfiar de facturas "perfectas" que llegan con contexto adicional armado (hilos de correo,
  aprobaciones simuladas) — es señal de un ataque más elaborado, no de mayor legitimidad.
- Revisar si las herramientas de seguridad del correo normalizan o descartan caracteres Unicode
  invisibles antes de aplicar sus reglas de detección.
- Entrenar a los equipos de cuentas por pagar específicamente en este patrón: no alcanza con
  "sospechar de links raros" cuando el ataque es una factura con formato impecable.

Ese último punto es el que más pesa. Ninguna herramienta reemplaza a una persona entrenada para
frenarse un segundo antes de aprobar un pago.

---

## Cómo te podemos ayudar

Este tipo de campaña está diseñada para pasar los filtros automáticos y llegar directo a la
bandeja de entrada de quien tiene el poder de aprobar un pago. La última barrera, en ese punto, es
la persona — y esa barrera se entrena.

En KŌGA hacemos **Capacitaciones en Phishing**: simulamos campañas realistas, medimos qué tan
expuesto está tu equipo y entrenamos específicamente los patrones que estafas como esta explotan,
desde impersonación ejecutiva hasta facturas fabricadas con IA. No es una charla genérica de
concientización — es entrenamiento contra el ataque que realmente les va a llegar.

→ [Conocé nuestro servicio de Capacitaciones en koga.ar](https://koga.ar/servicios#capacitaciones)

---

## Fuentes

- [KnowBe4 — AI-Assisted Phishing Campaign Sent Over a Million Personalized Emails](https://blog.knowbe4.com/ai-assisted-phishing-campaign-sent-over-a-million-personalized-emails)
- [KnowBe4 — Millions of Phishing Emails Use ASCII Smuggling to Bypass Security Filters](https://blog.knowbe4.com/millions-of-phishing-emails-use-ascii-smuggling-to-bypass-security-filters)
