---
layout: post
title: "The Gentlemen: el ransomware que mata 180 procesos antes de cifrar tu red"
date: 2026-08-01
categories: [news]
tags: [news, ransomware, raas, malware, go, byovd, edr-killer]
image:
  path: /assets/gentleman-raas.png
  alt: "The Gentlemen Ransomware"
author: c4cker
---

Desde julio de 2025, un grupo llamado **The Gentlemen** (también rastreado como Storm-2697) acumula víctimas a un ritmo sin precedentes entre operaciones de ransomware modernas: 48 ataques en enero de 2026, 91 en febrero, más de 332 confirmadas hasta junio. Es una operación RaaS con ~20 miembros, afiliados propios y un toolkit interno que liquida las defensas antes de que el cifrado empiece.

---

## Cómo funciona el ataque

El encryptor está escrito en Go, ofuscado con Garble, y requiere una contraseña para ejecutarse — complica el análisis en sandbox y evita detonaciones accidentales. Apunta a Windows, Linux, NAS, ESXi (locker dedicado en C) y entornos CSD.

Un ataque típico tiene tres fases antes de la nota de rescate.

---

### Fase 1 — Acceso inicial y reconocimiento (semanas antes del cifrado)

El grupo entra por vulnerabilidades conocidas en interfaces administrativas expuestas a internet: CVE-2024-55591 (FortiOS), CVE-2025-32433 (Erlang SSH) y CVE-2025-33073 (NTLM relay). Adentro, pasan entre dos y seis semanas haciendo reconocimiento, robando datos y preparando infraestructura antes de activar el ransomware. El cifrado es el paso final: cuando se detecta, la exfiltración ya terminó.

---

### Fase 2 — GentleKiller: borrar las defensas antes de empezar

Antes de cifrar, despliegan **GentleKiller**: framework propio de EDR-killing con más de 8 variantes BYOVD (Bring Your Own Vulnerable Driver). A diferencia de otros grupos, no dejan la evasión en manos de cada afiliado — distribuyen y mantienen GentleKiller centralizado, lo que da consistencia entre ataques y permite actualizarlo cuando un vendor parchea el driver abusado.

ESET documentó que apunta a más de 400 procesos de ~48 productos de seguridad (Microsoft Defender, CrowdStrike, Sophos, ESET) operando desde kernel-mode vía drivers legítimamente firmados pero vulnerables. La protección user-mode no alcanza para detenerlo.

Además de los procesos de seguridad, el encryptor termina cerca de 180 procesos y servicios: bases de datos, software de backup, aplicaciones empresariales — todo lo que mantiene archivos en uso.

---

### Fase 3 — Cifrado y autopropagación

Con el sistema limpio de defensas, el encryptor lanza autopropagación en paralelo sobre subredes adyacentes — concurrente con el cifrado, no una fase posterior.

El esquema criptográfico usa una clave efímera Curve25519 por archivo, cifrada con XChaCha20. Recuperar una clave no descifra las demás.

Antes de cifrar, borra evidencia forense: shadow copies (`vssadmin`/`wmic`), logs de Windows (System, Application, Security), Prefetch, logs de Defender e historial de RDP/PowerShell.

El movimiento lateral usa SystemBC (SOCKS5 con protocolo RC4 custom), Cobalt Strike, Mimikatz/comsvcs.dll para robo de credenciales y PsExec/WMI/WinRM/tareas programadas para propagación en Windows. Para Linux y ESXi, PuTTY vía SSH.

---

## El modelo de negocio

The Gentlemen no lanza campañas masivas esperando que una fracción pague: investigan y seleccionan targets manualmente, priorizando rescates significativos. La exfiltración previa sirve como segunda palanca — si no pagás, publican los datos.

En junio de 2026 el grupo sufrió un hackback: alguien accedió a su infraestructura y filtró datos operativos, analizados por Check Point Research.

---

## Remediación y detección

### Parchear los vectores de entrada

Tres CVEs explotados activamente como acceso inicial, todos con parches disponibles:

| CVE | Sistema |
|-----|---------|
| CVE-2024-55591 | FortiOS — interfaz de gestión |
| CVE-2025-32433 | Erlang SSH |
| CVE-2025-33073 | NTLM relay |

### IOC de aislamiento inmediato

`msimg32.dll` fuera de `C:\Windows\System32` es indicador de compromiso activo — aislar el host de inmediato.

### Contra GentleKiller y BYOVD

- Activar y mantener actualizada la **Microsoft Vulnerable Driver Blocklist** con HVCI habilitado.
- Habilitar **Memory Integrity** — bloquea carga de drivers no aprobados desde kernel-mode.
- Verificar que estén en blocklist los drivers abusados por GentleKiller: Safetica, Zemana, Qihoo 360, IObit y Huawei.
- EDR con **tamper protection a nivel de kernel**, no solo user-mode.
- **Memory capture en el EDR**: un dump durante cifrado activo puede recuperar las claves efímeras antes de que el malware las borre.

### Reducir superficie de movimiento lateral

- Restringir WMI remoto, WinRM, PsExec y tareas programadas remotas donde no sean necesarios.
- Bloquear herramientas de acceso remoto no autorizadas (AnyDesk, PuTTY) en endpoints.
- No exponer RDP, SSH ni consolas de gestión a internet — VPN con MFA como mínimo.
- Auditar y revocar `NullSessionShares`, `EveryoneIncludesAnonymous` y shares administrativos `$`: persisten aunque se limpie el malware si no se remedian explícitamente.
- Privilegios con tiempo acotado y auto-desescalada.

### Backups

El malware elimina shadow copies y logs antes de cifrar — backups conectados a la red son inútiles contra este grupo.

- Backups **air-gapped o con inmutabilidad garantizada** (S3 Object Lock, cinta offline).
- Probar restauración periódicamente.

### Detección — YARA y comportamiento

Check Point publicó una regla YARA para el locker. Las strings características son:

```
"Silent mode (don't rename files)"
"Encrypt only mapped and UNC network shares"
"README-GENTLEMEN.txt"
"gentlemen.bmp"
"gentlemen_system"
"[+] Encryption started"
```

Cuatro o más de estas strings en un PE = muestra confirmada. Reglas Sigma adicionales disponibles en SOC Prime.

Para detección por comportamiento:
- Proceso terminando 50+ procesos en ráfaga corta → alerta crítica.
- `vssadmin delete shadows` o `wevtutil cl System` → alerta + aislamiento automático del host.
- Driver cargado desde path fuera de `C:\Windows\System32` → posible abuso BYOVD.

---

## Fuentes

- [Microsoft Security Blog — Dissecting a self-propagating Go encryptor](https://www.microsoft.com/en-us/security/blog/2026/05/28/the-gentlemen-ransomware-dissecting-a-self-propagating-go-encryptor/)
- [Unit42 / Palo Alto Networks — No Manners Here: The Ruthless Rise](https://unit42.paloaltonetworks.com/the-gentlemen-ransomware/)
- [ESET WeLiveSecurity — Killing me gently: Inside GentleKiller](https://www.welivesecurity.com/en/eset-research/killing-me-gently-inside-gentlemens-edr-killer-framework/)
- [Halcyon — Scaling Faster Than Any Other Group on Record](https://www.halcyon.ai/ransomware-research-reports/threat-assessment-the-gentlemen-ransomware-group)
- [Check Point Research — When the Ransomware Gang Gets Hacked](https://blog.checkpoint.com/research/when-the-ransomware-gang-gets-hacked-what-the-gentlemen-leak-reveals-about-modern-ransomware-risk/)
- [Group-IB — Hastalamuerte TTPs Analysis](https://www.group-ib.com/blog/hastalamuerte-gentlemen-raas-ttps/)
- [Huntress — Defense Evasion TTPs](https://www.huntress.com/blog/the-gentlemen-ransomware-defense-evasion-ttps)
- [BleepingComputer — Multiple EDR killers](https://www.bleepingcomputer.com/news/security/gentlemen-ransomware-uses-multiple-edr-killers-to-disable-defenses/)
- [The Hacker News — GentleKiller targeting 400 processes](https://thehackernews.com/2026/06/the-gentlemen-raas-uses-gentlekiller.html)
- [Decryption Digest — 332 Victims via FortiGate RaaS](https://www.decryptiondigest.com/blog/gentlemen-ransomware-active-campaign-2026)
