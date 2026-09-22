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

Microsoft identificó una campaña que mandó más de un millón de correos personalizados a
departamentos de cuentas por pagar, usando IA generativa para fabricar facturas indistinguibles de
las reales. En paralelo, documentaron millones de correos que esconden palabras clave con
caracteres Unicode invisibles, rompiendo los filtros de detección sin que la víctima note nada raro
en pantalla. Dos técnicas distintas, mismo objetivo: que ni el filtro ni la persona del otro lado
detecten el engaño.

---

## Cómo funciona el ataque

La campaña combina varias capas de suplantación en un mismo correo: el nombre de un directivo real
en remitente, "responder a" y firma; facturas con branding, logos y una sección "Facturado a"
personalizada con el nombre real de la organización; y un hilo de conversación fabricado que le da
contexto a la factura para bajar la guardia. Las instrucciones de pago apuntan a cuentas distintas
según el objetivo, dificultando el rastreo.

El objetivo son transferencias de unos 50.000 dólares por víctima. El fraude de facturas no es
nuevo — la IA elimina el trabajo manual de armar cada pieza, permitiendo escalarlo a un millón de
correos sin perder personalización.

---

## El truco que esconde la palabra del filtro

La segunda técnica resuelve otro problema: cómo esconder palabras que dispararían una alerta. El
método inserta caracteres Unicode invisibles dentro de palabras clave como "funding". Para el ojo
humano la palabra se lee normal; para un filtro que busca coincidencias exactas, la secuencia de
bytes ya no coincide con nada conocido.

Los sistemas basados en coincidencia de texto o expresiones regulares no lo detectan. Tampoco los
clasificadores de machine learning, salvo que analicen una captura visual del mensaje en vez de
procesar el texto crudo. Microsoft reportó millones de correos con esta técnica — ya está adoptada
a escala.

Ninguna de las dos depende de malware ni de vulnerabilidades de software: dependen de que alguien
lea, confíe y actúe. La IA bajó el costo de producir el engaño perfecto; el Unicode invisible bajó
el riesgo de que quede atrapado en un filtro.

---

## Qué hacer con esto

- Verificar cambios de cuenta bancaria por un canal distinto al correo — llamada al número ya
  conocido del proveedor, no al que figura en el mensaje.
- Desconfiar de facturas "perfectas" con contexto adicional armado (hilos, aprobaciones simuladas):
  es señal de ataque elaborado, no de mayor legitimidad.
- Entrenar a cuentas por pagar específicamente en este patrón — no alcanza con "sospechar de links
  raros" cuando el ataque es una factura con formato impecable.

Ninguna herramienta reemplaza a una persona entrenada para frenarse un segundo antes de aprobar un
pago.

---

## Cómo te podemos ayudar

Este tipo de campaña está diseñada para pasar los filtros y llegar directo a quien tiene el poder
de aprobar un pago. La última barrera, ahí, es la persona — y esa barrera se entrena.

En KŌGA hacemos **Capacitaciones en Phishing**: simulamos campañas realistas y entrenamos
específicamente los patrones que estafas como esta explotan, desde impersonación ejecutiva hasta
facturas fabricadas con IA.

→ [Conocé nuestro servicio de Capacitaciones en koga.ar](https://koga.ar/servicios#capacitaciones)

---

## Fuentes

- [KnowBe4 — AI-Assisted Phishing Campaign Sent Over a Million Personalized Emails](https://blog.knowbe4.com/ai-assisted-phishing-campaign-sent-over-a-million-personalized-emails)
- [KnowBe4 — Millions of Phishing Emails Use ASCII Smuggling to Bypass Security Filters](https://blog.knowbe4.com/millions-of-phishing-emails-use-ascii-smuggling-to-bypass-security-filters)
