---
layout: post
title: "Medusa ya lleva más de 500 víctimas y sigue sumando"
date: 2026-08-19
categories: [news]
tags: [news, ransomware, medusa, raas, infraestructura-critica, cisa]
image:
  path: /assets/medusa-ransomware.png
  alt: "Medusa Ransomware — actualización agosto 2026"
author: c4cker
---

CISA, el FBI y el Departamento de Salud de EE.UU. actualizaron ayer la alerta conjunta sobre Medusa: el grupo de ransomware ya superó las 500 organizaciones de infraestructura crítica comprometidas desde 2021. La cifra veía 300 hace apenas un año y medio. El ritmo no está bajando.

---

## Ransomware, en criollo

Por si no lo tenés del todo claro: ransomware es el malware que cifra tus archivos y te pide un rescate para devolvértelos. La variante moderna — y Medusa es un caso de manual — no se conforma con eso: antes de cifrar, roba tus datos. Si no pagás por el descifrado, amenazan con publicarlos. Doble extorsión, le dicen. Pagás para recuperar o pagás para que no te expongan. A veces las dos cosas.

Medusa funciona como Ransomware-as-a-Service: un núcleo de desarrolladores arma el malware y lo alquila a afiliados, que se quedan con la mayor parte del rescate. Modelo franquicia, aplicado al cibercrimen.

---

## Cómo entran

Nada exótico. Dos rutas conocidas y bien documentadas:

**Vulnerabilidades sin parchear.** Los afiliados de Medusa explotan activamente CVE-2024-1709 (ConnectWise ScreenConnect) y CVE-2023-48788 (Fortinet FortiClient EMS) — ambas con parche disponible desde hace más de un año. Si tu organización todavía las tiene sin corregir, sos un blanco fácil, no un blanco sofisticado.

**Corredores de acceso inicial.** Medusa recluta "brokers" en foros de cibercrimen que ya tienen credenciales robadas o accesos comprados. Pagan entre 100 dólares y 1 millón, según la calidad del acceso. El phishing sigue siendo la puerta de entrada más barata para conseguir esas credenciales.

Una vez adentro, usan herramientas legítimas para pasar desapercibidos: Advanced IP Scanner y SoftPerfect para mapear la red, AnyDesk y PsExec para moverse lateralmente, Mimikatz para robar credenciales, rclone para sacar los datos antes de cifrar todo. Nada de eso dispara una alarma de antivirus tradicional — son herramientas que cualquier administrador de sistemas también usa.

---

## Quién está en la mira

Salud pública, base industrial de defensa, manufactura crítica, gobierno, tecnología y servicios financieros. No son sectores elegidos al azar: son los que peor pueden permitirse un sistema caído y, muchas veces, los que más lento actualizan su infraestructura.

---

## Qué hacer con esto

Ninguna de las dos rutas de entrada requiere magia para cerrarse:

- Parchear ScreenConnect y FortiClient EMS si los tenés en producción — y en general, mantener al día cualquier sistema con acceso remoto expuesto.
- Segmentar la red. Si un afiliado entra por una máquina, que no pueda caminar libremente hasta el resto.
- Restringir accesos remotos desde orígenes no confiables — RDP y VPN sin control son la alfombra roja de este tipo de grupos.
- Vigilar el uso de herramientas legítimas fuera de su patrón habitual. Un PsExec o un rclone corriendo donde no debería es la señal más temprana que vas a tener.

Ese último punto es, en la práctica, el más difícil de sostener sin ayuda: requiere ojos puestos en la red las 24 horas, no una revisión semanal.

---

## Cómo te podemos ayudar

En KŌGA identificamos si tu organización tiene expuestas las mismas puertas que Medusa ya explotó en cientos de víctimas — con nuestro servicio de [Pentesting](https://koga.ar/servicios#pentesting) evaluamos vulnerabilidades conocidas sin parchear y accesos remotos mal configurados. Y con **ZEN**, nuestro servicio de monitoreo 24/7, detectamos ese uso anómalo de herramientas legítimas — el momento exacto donde todavía se puede frenar un ataque antes de que llegue al cifrado.

→ [Conocé nuestros servicios en koga.ar](https://koga.ar/servicios)

---

## Fuentes

- [CISA — #StopRansomware: Medusa Ransomware (actualización agosto 2026)](https://www.cisa.gov/news-events/cybersecurity-advisories/aa25-071a)
- [BleepingComputer — CISA: Medusa ransomware hit over 500 critical infrastructure orgs](https://www.bleepingcomputer.com/news/security/cisa-medusa-ransomware-hit-over-500-critical-infrastructure-orgs/)
- [Industrial Cyber — US exposes Medusa ransomware threat](https://industrialcyber.co/cisa/us-exposes-medusa-ransomware-threat-as-over-300-organizations-targeted-across-critical-infrastructure-sector/)
