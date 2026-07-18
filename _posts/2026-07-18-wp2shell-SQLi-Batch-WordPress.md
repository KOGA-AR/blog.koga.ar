---
layout: post
title: "CVE-2026-63030 + CVE-2026-60137: de SQLi ciega a RCE en WordPress, reproducido en lab"
date: 2026-07-18
categories: [pentesting, web]
tags: [sqli, wordpress, sql-injection, wp2shell, docker, blind-sqli, rce, cve-2026-63030, cve-2026-60137]
author: c4cker
---

Esta semana WordPress lanzó parches de emergencia (6.9.5 y 7.0.2, el 17/07/2026, con auto-update
forzado) para una cadena de dos vulnerabilidades: una confusión de rutas en el endpoint REST "batch"
(**CVE-2026-63030**, reportada por Adam Kues de Assetnote/Searchlight Cyber) que habilita una SQL
injection sin autenticación en `WP_Query` (**CVE-2026-60137**, reportada por TF1T, dtro y haongo).
Searchlight, que descubrió el primer bug, no publicó el detalle técnico del salto final a RCE dada
la severidad. Desde KOGA armamos un lab local con Docker para reproducir la cadena con
[wp2shell-poc](https://github.com/Icex0/wp2shell-poc), un PoC público que cubre la SQLi ciega y,
con credenciales de admin ya obtenidas, el salto a RCE vía un plugin webshell.

Aclaración necesaria: lo que reprodujimos — SQLi ciega → hash → cracking → login admin → webshell —
**no es el 0-day sin credenciales** que insinúan algunos titulares. Es el camino del PoC público,
probablemente el mismo que siguió Searchlight antes de guardarse el paso final que sí lograron sin
autenticación.

Setup completo del lab (Docker Compose + troubleshooting): [wp2shell](https://github.com/KOGA-AR/wp2shell).

---

## La vulnerabilidad

WordPress soporta desde la 5.6 "batch requests" en su REST API: un POST a `/wp-json/batch/v1` (o
`/?rest_route=/batch/v1` sin pretty permalinks) con un array `requests` que el core despacha
internamente como llamadas independientes.

**CVE-2026-63030** es una confusión de rutas en `serve_batch_request_v1()`: un sub-request malformado
(tipo `http://:`) desincroniza los arrays internos del dispatcher (`$requests`/`$validation`/
`$matches`), y un GET a `/wp/v2/users` termina ejecutándose en un contexto de validación que no le
corresponde. Ese contexto "prestado" es lo que deja pasar sin sanear el parámetro `author_exclude`
hasta `WP_Query` — ahí es donde pega **CVE-2026-60137**, la SQLi real, en la cláusula:

```
... post_author NOT IN (<value>) ...
```

Inyección clásica en `NOT IN`, ciega por timing (`SLEEP()`), sin sesión ni nonce.

**Versiones:** CVE-2026-63030 en 6.9.0–6.9.4 y 7.0.0–7.0.1; CVE-2026-60137 también en 6.8.0–6.8.5
(pero ahí no es alcanzable sin auth, falta la confusión de rutas). **Parches:** 6.9.5, 7.0.2 y
backport a 6.8.6, todos el 17/07/2026, con auto-update forzado.

El PoC público que usamos cubre `check`/`read` (SQLi ciega, sin auth) y `shell` — pero este último
**requiere credenciales de admin ya obtenidas**, no es el 0-day sin credenciales que reportaron
algunos medios generalistas al hablar de "RCE" sin matizar.

---

## El lab

Dos instancias ancladas a versión exacta con Docker Compose, cada una en su red aislada y bindeada
solo a `127.0.0.1`: `wordpress:6.9.4-php8.2-apache` (puerto 8096) y `wordpress:7.0.1-php8.3-apache`
(puerto 8097), cada una con su MariaDB sin exponer puertos al host.

Cuatro problemas propios que nos costaron más tiempo que la SQLi:

- **`internal: true` en la red rompe el port-forward.** Sin gateway, Docker no puede publicar
  puertos al host — el bind a `127.0.0.1` deja de funcionar ("Connection refused"). El bind a
  loopback ya es suficiente aislamiento; para bloquear salida a internet del contenedor, usar
  `iptables`/`ufw` en el host, no tocar la red del compose.
- **Cambiar la red sin recrear el stack rompe el DNS interno.** Al volver `internal: false`,
  WordPress tiraba `getaddrinfo for db failed`. Fix: `down` (sin `-v`) + `up -d` para que ambos
  contenedores se reconecten a una red nueva.
- **Permalinks "Plano" rompen `/wp-json/...`.** 404 de Apache, no de WordPress. El PoC ya lo
  contempla con `--rest-route`, que usa el fallback `?rest_route=`.
- **El hash `$wp$2y$10$...` no es `phpass` clásico.** Desde WP 6.8 es bcrypt nativo envuelto con
  prefijo `$wp$`. Para hashcat modo `3200` hay que reemplazar `$wp$` por un solo `$` (no borrarlo
  entero, o el hash queda en 59 caracteres en vez de 60 y da "Token length exception").

---

## Reproduciendo la cadena

```bash
python wp2shell.py check --rest-route http://127.0.0.1:8096
python wp2shell.py check --rest-route --confirm-sqli http://127.0.0.1:8096
python wp2shell.py read --rest-route --preset users http://127.0.0.1:8096
python wp2shell.py shell http://127.0.0.1:8096 --user admin --password '<password>' --cmd id
```

En 6.9.4, el batch probe confirma route-confusion (`parse_path_failed`, `block_cannot_read`,
`rest_batch_not_allowed`), la confirmación por timing muestra baseline 0.04s vs. inyectado 3.06s
(threshold 1.95s), y `read` extrae el usuario con su hash:

```
[+] 1|admin|$wp$2y$10$IIA5UGa1YDKJmjjQyRrsBu69/2jutTvONgL0In9NvZnoT1GrR0pFS
```

Con la contraseña conocida, `shell` autentica, sube el plugin webshell y ejecuta comandos:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

En 7.0.1 corrimos la misma secuencia con resultado idéntico — mismos markers, mismo patrón de
timing (~3s sobre baseline ~0.05s), otro usuario (`admin2`) con su hash, y RCE final como
`www-data`. La consistencia entre versiones confirma que es el mismo bug, sin cambios relevantes
entre esos releases.

---

## Cracking del hash

```bash
echo '$wp$2y$10$IIA5UGa1YDKJmjjQyRrsBu69/2jutTvONgL0In9NvZnoT1GrR0pFS' \
  | sed 's/^\$wp\$/\$/' > hash_bcrypt.txt   # $2y$10$... (60 chars, bcrypt válido)

hashcat -m 3200 hash_bcrypt.txt /usr/share/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

Bcrypt es lento por diseño, así que el tiempo de cracking depende de qué tan arriba esté la
contraseña real en el diccionario — nada que ver con MD5/SHA1 en GPU. En un entorno real con
contraseñas fuertes, este paso es el cuello de botella: aun con la SQLi confirmada, sin una
contraseña débil el atacante no llega al RCE final.

---

## Mitigación

- **Actualizar ya** a 6.8.6 / 6.9.5 / 7.0.2 — WordPress ya forzó el auto-update, pero conviene
  confirmarlo si tenés esa opción deshabilitada.
- Bloquear `/wp-json/batch/v1` y `/?rest_route=/batch/v1` a nivel de proxy si no usás batch
  requests (Wordfence ya desplegó una regla de WAF para sus clientes el mismo 17/07).
  Requests con sub-requests malformados (`http://:`, `///`) hacia ese endpoint son la firma del
  ataque.
- Contraseñas de admin con entropía suficiente siguen siendo la última línea de defensa real: el
  algoritmo (bcrypt) es sólido, el riesgo está en la contraseña elegida.

---

## Remediación (si sospechás que ya te explotaron)

Actualizar corta la vía de entrada, pero no deshace nada que haya pasado antes del parche. Si tu
sitio estuvo en el rango afectado y expuesto públicamente entre la introducción del bug y el
17/07/2026, conviene tratarlo como potencialmente comprometido y no solo como "ya parcheado":

- **Revisar los logs de acceso** en busca de POSTs a `/wp-json/batch/v1` o `/?rest_route=/batch/v1`,
  sobre todo con sub-requests anidados o paths malformados (`http://:`, `///`). Son la firma del
  probing, con o sin éxito.
- **Auditar `wp_users`** por cuentas de administrador que no reconozcas, sobre todo creadas después
  de la fecha del primer acceso sospechoso en los logs.
- **Revisar `wp-content/plugins/`** por carpetas de plugin no instaladas conscientemente — el patrón
  típico de esta cadena es un directorio con nombre aleatorio tipo `wp2shell_<hex>` o similar,
  conteniendo un único archivo PHP.
- **Rotar credenciales y secretos**: contraseñas de todos los usuarios con rol admin, y los `SALT`/
  `KEY` de `wp-config.php` (`AUTH_KEY`, `SECURE_AUTH_KEY`, `LOGGED_IN_KEY`, `NONCE_KEY` y sus pares
  `_SALT`), lo que invalida además todas las sesiones activas.
- **Buscar archivos modificados recientemente** fuera de lo esperado (`find wp-content -mtime -N
  -name "*.php"`), y comparar el core contra una instalación limpia de la misma versión si hay dudas
  de integridad.
- **Revisar cron jobs** (`wp_options` → `cron`, y crontabs del sistema si hay acceso a shell) por
  tareas programadas que no correspondan.
- Si hay evidencia concreta de compromiso (plugin desconocido, usuario admin no reconocido,
  archivos modificados), la opción más segura es restaurar desde un backup previo a la ventana de
  exposición y recién ahí aplicar el parche, en vez de parchear sobre una instalación potencialmente
  comprometida.

---

## Conclusión

Más allá de la SQLi, lo que más nos dejó este ejercicio fueron los tropiezos de infraestructura:
networking de Docker, el cambio de esquema de hasheo de WordPress, y la diferencia entre "el
endpoint responde" y "la vulnerabilidad específica existe". Repo completo del lab, con todos estos
problemas documentados: [wp2shell](https://github.com/KOGA-AR/wp2shell).

Si administrás o auditás infraestructura WordPress y necesitás validar exposición a esta cadena o a
otras vulnerabilidades de tu stack, es el tipo de trabajo que hacemos en KOGA en nuestros
engagements de pentesting web.

---

## Fuentes

- [GHSA-ff9f-jf42-662q — CVE-2026-63030](https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-ff9f-jf42-662q)
- [GHSA-fpp7-x2x2-2mjf — CVE-2026-60137](https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-fpp7-x2x2-2mjf)
- [WordPress 7.0.2 Security Release](https://wordpress.org/news/2026/07/wordpress-7-0-2-release/)
- Searchlight Cyber / Assetnote — research post del descubrimiento (sin detalle técnico del RCE)
- Análisis técnico de la confusión de rutas — Hadrian.io
- [wp2shell-poc](https://github.com/Icex0/wp2shell-poc) — PoC público usado en este post
