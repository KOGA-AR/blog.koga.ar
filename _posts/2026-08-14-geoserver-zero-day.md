---
layout: post
title: "Zero-day en GeoServer: explotación activa sin parche disponible"
date: 2026-08-14
categories: [news]
tags: [news, zero-day, geoserver, rce, sql-injection, gis, infraestructura-critica]
image:
  path: /assets/geoserver-zero-day.png
  alt: "Zero-day en GeoServer — agosto 2026"
author: c4cker
---

El martes 12 de agosto, un investigador publicó en X los detalles de una vulnerabilidad de inyección SQL en GeoServer — el servidor de mapas open source que usan gobiernos, municipios y organizaciones de infraestructura crítica en todo el mundo. Horas después, los intentos de explotación ya estaban en curso. Al momento de publicar este post, no existe un parche disponible.

---

## Qué es GeoServer y por qué importa

GeoServer es el servidor estándar para publicar datos geoespaciales: catastros provinciales, portales de planificación territorial, mapas de infraestructura pública, sistemas de monitoreo ambiental. Lo usan organismos que raramente aparecen en conversaciones de ciberseguridad pero que sostienen servicios críticos.

En Argentina, varios organismos provinciales y nacionales lo tienen desplegado. Tucumán tiene GeoServer activo en su [sistema de información territorial IDET/GeoSPlan](http://geosplan.tucuman.gov.ar/). El Ministerio de Ambiente lo usa para datos de suelo de la región NOA. Salta tiene su geoportal catastral vinculado a servicios WFS/WMS. No son los únicos.

---

## El zero-day: SQL injection en `jsonArrayContains`

El investigador @q1uf3ng identificó una falla en la función `jsonArrayContains` de GeoServer — un filtro para consultar campos JSON en bases de datos. El problema es clásico: los argumentos que recibe del usuario no se sanitizan correctamente antes de ser codificados en la consulta a la base de datos.

Resultado: inyección SQL. Y bajo ciertas configuraciones — específicamente cuando el data store es **PostGIS** u **Oracle JDBC** y la base de datos corre con cuenta de administrador — la inyección SQL escala a ejecución remota de código en el servidor.

No hay CVE asignado. No hay parche. El researcher publicó los detalles en abierto el 12 de agosto.

## Explotación activa desde el primer día

La firma watchTowr registró intentos de explotación dentro de las horas siguientes a la divulgación. Jake Knott, investigador principal, describió la actividad como reconocimiento: los atacantes están escaneando para identificar instancias vulnerables expuestas, provocando errores para confirmar si el endpoint responde.

Su advertencia: *"Es poco probable que esto siga así por mucho tiempo. GeoServer tiene un historial de ser atacado y explotado a escala."*

El historial es concreto. En 2024, CVE-2024-36401 (CVSS 9.8, también RCE en GeoServer) fue explotado por grupos APT vinculados a China para comprometer organismos de gobierno en Asia y una agencia federal de EE.UU. CISA lo agregó a su catálogo de vulnerabilidades explotadas activamente. El grupo APT41 usó esa falla para desplegar el backdoor SideWalk.

GeoServer no es un objetivo nuevo. Es un objetivo recurrente.

---

## El problema específico del sector público argentino

La mayoría de los portales GIS gubernamentales están expuestos a internet por diseño — su función es ser accesibles al público. Eso significa que la superficie de ataque está, en muchos casos, directamente en internet sin capas adicionales de protección.

La pregunta concreta: ¿qué instancias de GeoServer argentinas están corriendo con PostGIS o Oracle JDBC, expuestas públicamente, sin restricciones de acceso al endpoint `jsonArrayContains`?

La respuesta honesta es que la mayoría de los organismos afectados no lo saben todavía.

---

## Qué hacer ahora

Sin parche disponible, las opciones son de contención:

**Identificar instancias expuestas.** Auditar qué servidores GeoServer están publicados hacia internet. Buscar en inventarios, en infraestructura de proveedores, en los servicios que el organismo publica para acceso público.

**Restringir acceso al endpoint vulnerable.** Si el servicio no necesita estar abierto a internet pública, no debería estarlo. WAF o reglas de red que bloqueen o restrinjan requests al endpoint `jsonArrayContains` mientras se espera el parche oficial.

**Monitorear actividad anómala.** Requests inusuales contra los endpoints de filtro de GeoServer, errores en la capa de base de datos, consultas que no corresponden al uso normal del sistema.

**Seguir el repositorio oficial.** El proyecto GeoServer publicará el parche. Aplicarlo en cuanto esté disponible — la ventana entre disclosure y explotación masiva ya se abrió.

---

## ¿Tenés GeoServer en tu infraestructura?

En KŌGA ayudamos a organismos y empresas a identificar exposición en servicios como GeoServer antes de que lo haga alguien más: mapeamos instancias expuestas, evaluamos la configuración de data stores y revisamos si el tráfico hacia esos endpoints tiene los controles necesarios.

Si tenés GeoServer en producción — o si no sabés con certeza si tu infraestructura lo tiene — es el momento de revisarlo.

→ [Contactanos en koga.ar](https://koga.ar/)

---

## Fuentes

- [SecurityWeek — Hackers Exploiting Unpatched GeoServer Zero-Day](https://www.securityweek.com/hackers-exploiting-unpatched-geoserver-zero-day/)
- [The Hacker News — Unpatched GeoServer Zero-Day Targeted in Active Exploitation Attempts](https://thehackernews.com/2026/08/unpatched-geoserver-zero-day-targeted.html)
- [The Hacker News — GeoServer Vulnerability Targeted to Deliver Backdoors (CVE-2024-36401)](https://thehackernews.com/2024/09/geoserver-vulnerability-targeted-by.html)
- [Fortinet — Threat Actors Exploit GeoServer CVE-2024-36401](https://www.fortinet.com/blog/threat-research/threat-actors-exploit-geoserver-vulnerability-cve-2024-36401)
