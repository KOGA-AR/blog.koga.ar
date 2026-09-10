---
layout: post
title: "CVE-2026-85102 y CVE-2026-85103: RCE sin autenticación en VPN de Check Point"
date: 2026-09-10
categories: [news, pentesting]
tags: [news, vpn, check-point, rce, cve-2026-85102, cve-2026-85103, perimetro, pentesting-externo]
image:
  path: /assets/checkpoint-vpn-rce.png
  alt: "Check Point VPN RCE"
author: c4cker
---

Check Point divulgó esta semana dos vulnerabilidades críticas en el procesamiento de certificados
VPN de sus Security Gateways, ambas con CVSS 9.8. Las dos permiten ejecución remota de código sin
autenticación. Los parches de emergencia ya están disponibles — pero el tiempo entre divulgación
y explotación activa en este tipo de productos se mide en días.

---

## Las vulnerabilidades

**CVE-2026-85102** es un fallo en la validación de confianza de certificados durante la
negociación VPN. El gateway acepta un certificado presentado por un atacante sin verificar
correctamente si es de confianza, lo que permite llevar la negociación lo suficientemente lejos
como para ejecutar código arbitrario en el equipo. Afecta tanto configuraciones de Remote Access
VPN como Site-to-Site.

**CVE-2026-85103** es un heap-based buffer overflow en el decoder ASN.1 que el gateway usa para
procesar la estructura del certificado. El overflow ocurre antes de cualquier verificación de
autenticidad, lo que convierte cualquier conexión VPN entrante en un vector potencial. Afecta
Quantum Security Gateway y Quantum Security Management.

Ambas requieren condiciones específicas para ser explotadas, pero ninguna requiere credenciales ni
acceso previo al sistema. Las encontró el propio equipo de investigación de Check Point — sin
evidencia de explotación activa ni PoC público al momento de escribir esto.

---

## Por qué importa en un pentest externo

Un gateway de Check Point expuesto a internet con una de estas dos vulnerabilidades es RCE desde
afuera, sin credenciales. En términos de pentest externo, es el peor hallazgo posible en un
dispositivo de perímetro: acceso completo al appliance que controla el tráfico de red de la
organización, sin necesidad de phishing, sin credencial comprometida, sin pasar por ningún portal.

Check Point Quantum Security Gateway es uno de los firewalls/VPN más desplegados en entornos
corporativos medianos y grandes. Si aparece en superficie de ataque externa sin el parche aplicado,
aparece en el reporte.

---

## Qué hacer ahora

Check Point lanzó hotfixes de emergencia para ambas CVEs. Los productos afectados son:

- Quantum Security Gateway (todas las versiones soportadas)
- Quantum Security Management
- Spark Firewall

**Actualizar es la única mitigación real.** Mientras no se aplique el parche:

- Verificar si el servicio VPN está expuesto directamente a internet o solo detrás de otro
  control de acceso.
- Monitorear logs del gateway por negociaciones VPN con certificados desconocidos o de orígenes
  inesperados.
- Si el appliance no necesita Remote Access VPN habilitado, deshabilitar el servicio reduce la
  superficie expuesta.

---

## ¿Sabés qué tiene expuesto tu organización?

Este tipo de vulnerabilidad aparece constantemente en assessments externos: dispositivos de
perímetro con versiones sin parchear, expuestos a internet sin saberlo. El problema no es solo
Check Point — es no tener visibilidad sobre qué superficie real presenta tu organización hacia
afuera.

> **En KOGA hacemos pentesting externo:** mapeamos tu superficie de ataque, identificamos
> dispositivos y servicios expuestos, y validamos qué es explotable antes de que lo haga alguien
> con intenciones reales. Si usás Check Point o cualquier otro gateway VPN y no tenés certeza de
> tu exposición, [hablemos](https://koga.ar/servicios#pentesting).

---

## Fuentes

- [The Hacker News — Check Point Discloses Two 9.8-Rated VPN Certificate Flaws](https://thehackernews.com/2026/09/check-point-discloses-two-98-rated-vpn.html)
- [CyberSecurityNews — Critical Check Point VPN Vulnerabilities Enable RCE](https://cybersecuritynews.com/check-point-vpn-vulnerabilities/)
- [Check Point Blog — Hotfix for Vulnerabilities in Deprecated IKEv1 VPN Protocol](https://blog.checkpoint.com/security/check-point-releases-important-hotfix-for-vulnerabilities-in-deprecated-ikev1-vpn-protocol/)
- [SOCRadar — CVE-2026-50751: Check Point VPN Auth Bypass](https://socprime.com/blog/cve-2026-50751-check-point-vpn-authentication-bypass-exploited-in-targeted-attacks/)
- [SecurityOnline — CVE-2026-85102 & 85103: Check Point VPN Flaws CVSS 9.8](https://securityonline.info/checkpoint-vpn-vulnerabilities/)
