---
layout: post
title: "No necesitaron un exploit. Usaron las credenciales que venían de fábrica."
date: 2026-08-11
categories: [news]
tags: [news, ics, ot-security, scada, sandworm, industrial-security, plc, siemens, apt]
image:
  path: /assets/poland-power-plant.png
  alt: "Ataque a planta de energía en Polonia — diciembre 2025"
author: c4cker
---

En diciembre de 2025, atacantes vinculados al grupo ruso Sandworm tomaron control remoto de una planta de calefacción combinada en Polonia. Detuvieron una turbina de vapor. Borraron el acceso a los sistemas de control industrial. Y lo hicieron sin explotar ninguna vulnerabilidad conocida: sin CVE, sin zero-day, sin malware sofisticado.

---

## La cadena de ataque: once días invisibles

Ruta real: granja eólica → APN privada → planta de calor.

**Red APN** (*Access Point Name*) privada permitía tráfico cliente-a-cliente entre cualquier dispositivo. Privada no significa aislada.

**18 de diciembre:** atacantes escanearon APN desde granja eólica comprometida (FortiGate sin MFA, acceso de admin).

**25 de diciembre:** conectaron a PLCs Siemens S7-300/1200/1500 vía protocolo S7 nativo. Reconocimiento sin malware — no necesitaban código, solo credenciales de administrador por defecto del WAGO PFC200 y acceso remoto sin restricciones.

**29 de diciembre, 5:30 AM:** pararon turbina de vapor, apagaron tratamiento de agua, resetearon servidores Moxa a factory defaults y les asignaron IPs inalcanzables (127.0.0.1). 

**7:30 AM:** comenzó recuperación con atacante aún adentro.

Once días entre escaneo y ataque destructivo. Tiempo suficiente para detectar. Nadie monitoreaba la APN.

---

## Primera instancia conocida: APN privada como vector OT

CERT Polska documentó esto como primer ataque real usando APN privada para llegar a infraestructura industrial.

ESET atribuyó a **Sandworm** (grupo ruso, GRU) con confianza media — TTPs solapan con apagones en Ucrania 2015/2016 y NotPetya.

Punto clave: técnicas usadas no requieren ser un APT ruso. Pivotar por APN, explotar credenciales por defecto, hablar protocolos industriales. Ninguna sofisticada. Efectivas porque nadie las mapea como vectores.

---

## Tres preguntas para hoy

**¿Qué sistemas tienen conectividad externa sin documentar?**

APN privadas, módems industriales, dispositivos de monitoreo remoto de proveedores — puntos ciegos frecuentes. Instalados por mantenimiento o terceros, no en inventario de TI. No mapeado = no defendido.

Cómo revisarlo: pregunta al equipo de operaciones qué equipos de campo tienen conexión remota. Pregunta a proveedores qué dispositivos dejaron instalados para monitoreo. Si no hay respuesta clara, es un punto ciego.

**¿Acceso remoto a controles sin MFA?**

FortiGate en Polonia: credenciales locales sin segundo factor. Quienquiera que tuviera esas claves entraba como admin. El atacante no necesitó phishing, no necesitó vulnerabilidades — la contraseña era la que vino configurada.

Revisar: audita VPNs, interfaces de gestión remota, SSH en equipos industriales. Si aceptan credenciales sin segundo factor, son puertas de entrada.

**¿Red OT realmente aislada, o solo separada en papel?**

APN con tráfico cliente-a-cliente = sin barrera. No hay firewall que romper para pasar de granja eólica a planta de control. Separación lógica ≠ aislamiento real.

Verificar: si un equipo comprometido en la red de monitoreo puede alcanzar los PLCs y controladores, la red no está aislada — está dividida geográficamente, no técnicamente.

---

## Detectar antes de que el daño sea irreversible

En Polonia tuvieron once días: del 18 al 29 de diciembre. Escaneo, reconocimiento, preparación, luego el golpe a las 5:30 AM. Suficiente para detectar si alguien hubiera estado monitoreando tráfico anómalo en la APN.

Las organizaciones que contienen este tipo de ataques tienen dos cosas en común:

1. **Visibilidad en redes "privadas"** — monitorizan APN, redes de proveedores, VLANs de mantenimiento, no solo la red corporativa.
2. **Saben qué debería estar conectado** — si aparece un escaneo de puertos en la APN, si alguien intenta conectarse a un PLC desde una dirección inesperada, eso es anomalía detectable.

Lo que casi ninguna organización tiene es ese mapeo inicial: *"estos son todos nuestros sistemas de control, esto es quién debería acceder a ellos, esto es cómo debería verse el tráfico normal"*.

Sin ese mapa, el primer ataque es siempre el que te sorprende.

---

## Si tu organización tiene infraestructura física conectada

En KŌGA ayudamos empresas a entender cuál es su exposición real. No es auditoría teórica: es mapa práctico de qué está expuesto, quién puede alcanzarlo, cómo detectar cuándo algo sale de lo normal.

Hacemos:

* **Asset discovery** en redes OT, APN, redes de proveedores — todo lo que no está en el inventario oficial.
* **Monitoreo continuo** 24/7 para detectar comportamiento anómalo antes de que cause daño.
* **Auditoría de acceso remoto** — VPNs, interfaces de gestión, credenciales, segundo factor.
* **Entrenamiento de equipos** — cómo reconocer intentos de acceso remoto no autorizados.

Si tu empresa tiene turbinas, compresores, sistemas de tratamiento, líneas de producción, subestaciones, o cualquier máquina crítica conectada a la red, y nunca realizó una evaluación de seguridad OT, este es el momento.

→ [Contactanos en koga.ar](https://koga.ar/) — mapear tu exposición antes de que alguien más lo haga.

---

## Fuentes

- [The Hacker News — Hackers Breach Polish Power Plant Controls via Private Cellular Network](https://thehackernews.com/2026/08/hackers-breach-polish-power-plant.html)
- [CERT Polska — Energy Sector Incident Report, December 2025](https://cert.pl/en/posts/2026/01/incident-report-energy-sector-2025/)
- [BleepingComputer — Hackers breached a small Polish energy plant via private APN](https://www.bleepingcomputer.com/news/security/hackers-breached-a-small-polish-energy-plant-via-private-apn-last-year/)
- [SecurityWeek — Novel Private APN Pivot Let Hackers Sabotage Second Polish Energy Facility](https://www.securityweek.com/novel-private-apn-pivot-let-hackers-sabotage-second-polish-energy-facility/)
- [Security Affairs — Hackers Cross From IT to OT Through a Private APN in Poland](https://securityaffairs.com/196955/security/hackers-cross-from-it-to-ot-through-a-private-apn-in-poland.html)
- [Industrial Cyber — CERT Polska details cyberattacks on Polish manufacturer, energy sites](https://industrialcyber.co/industrial-cyber-attacks/cert-polska-details-cyberattacks-on-polish-manufacturer-energy-sites-fails-to-disrupt-power-and-heat-supply/)
