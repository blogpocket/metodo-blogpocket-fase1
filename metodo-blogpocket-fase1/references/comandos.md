# Llamadas exactas

Todo pasa por las herramientas de WPVibe. Los nombres de los parámetros de cada
herramienta se cargan con `tool_search` al empezar (`site_info`, `run_wp_cli`,
`rest_api`, `get_page_html`, `audit_page`, `show_approval_panel`); lo que hay aquí es el
**contenido** de cada llamada, que es la parte que falla si se escribe de otra manera.

Todas las órdenes de WP-CLI van en el parámetro `command` de `run_wp_cli`, con
`site_url` apuntando al sitio. En las consultas a la base de datos, `{prefix}` es el
marcador del prefijo de tablas: déjalo tal cual, WPVibe lo sustituye.

## Índice

1. Reconocimiento del sitio
2. Paso 1 · LiteSpeed Cache
3. Paso 2 · Loginizer fuera, Security Optimizer dentro
4. Paso 3 · Fuentes del sistema
5. Medición

---

## 1. Reconocimiento del sitio

```
db query "SELECT post_type, post_status, COUNT(*) FROM {prefix}posts WHERE post_type IN ('post','page') GROUP BY post_type, post_status"
```

```
db query "SELECT ID, post_name FROM {prefix}posts WHERE post_type = 'wp_global_styles'"
```

Sin filas en la segunda consulta, el tema activo no es de bloques y el paso 3 no aplica.

---

## 2. Paso 1 · LiteSpeed Cache

Instalación en dos tiempos. La primera llamada devuelve qué versión se instalaría; la
segunda, idéntica pero con `confirm_write` activado, la instala.

```
plugin install litespeed-cache --activate
```

Los tres ajustes. El formato importa: `--format=plaintext` en los tres, y el valor de
`cache-exc` es una lista JSON entre comillas simples.

```
option update litespeed.conf.cache-exc --format=plaintext '["/wp-json/"]'
option update litespeed.conf.cache-rest --format=plaintext 0
option update litespeed.conf.cache-browser --format=plaintext 1
```

Verificación. LiteSpeed guarda cada ajuste como una opción suelta con el prefijo
`litespeed.conf.`, no como un array serializado:

```
db query "SELECT option_name, option_value FROM {prefix}options WHERE option_name IN ('litespeed.conf.cache','litespeed.conf.cache-exc','litespeed.conf.cache-rest','litespeed.conf.cache-browser','litespeed.conf.cache-ttl_pub','litespeed.conf.cache-ttl_browser')"
```

Para auditar la configuración entera en cualquier momento:

```
db query "SELECT option_name, option_value FROM {prefix}options WHERE option_name LIKE 'litespeed.conf.%'"
```

Purga de caché (no acepta argumentos adicionales):

```
cache purge
```

---

## 3. Paso 2 · Loginizer fuera, Security Optimizer dentro

**Por su archivo, no por su carpeta.** `plugin deactivate loginizer` falla en silencio.

```
plugin deactivate loginizer/loginizer.php
plugin uninstall loginizer/loginizer.php
```

La desinstalación devuelve un `op_id` y exige aprobación: pásalo a
`show_approval_panel` y espera a que la persona confirme en el navegador.

Comprobación de restos en los cuatro sitios. Las tres primeras son consultas; la cuarta,
el listado de tareas programadas.

```
db query "SELECT option_name FROM {prefix}options WHERE option_name LIKE '%loginizer%'"
db query "SELECT meta_key FROM {prefix}usermeta WHERE meta_key LIKE '%loginizer%'"
db query "SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME LIKE '%loginizer%'"
cron event list
```

Lo habitual es que sobreviva una opción. Se borra a mano, y también pide aprobación:

```
option delete loginizer_pro_less_than_201
```

Instalación de Security Optimizer, otra vez en dos tiempos:

```
plugin install sg-security --activate
```

Verificación de las siete protecciones de fábrica:

```
db query "SELECT option_name, option_value FROM {prefix}options WHERE option_name LIKE 'sg_security%'"
```

Las siete que tienen que estar, todas a `1` salvo la última:

| Opción | Valor |
|---|---|
| `sg_security_lock_system_folders` | 1 |
| `sg_security_disable_file_edit` | 1 |
| `sg_security_disable_xml_rpc` | 1 |
| `sg_security_wp_remove_version` | 1 |
| `sg_security_xss_protection` | 1 |
| `sg_security_disable_usernames` | 1 |
| `sg_security_login_attempts` | 5 |

Comprobación a mano de la etiqueta `generator` (purga la caché antes):

```
get_page_html  →  url: la portada,  selector: head,  max_length: 2000
```

Si aparece `<meta name="generator" content="WordPress ...">`, la protección de ocultar
la versión no está surtiendo efecto. Anótalo en el informe.

La dirección de las notificaciones semanales queda en la que WordPress inventa durante
la instalación (`admin@` más el dominio), que en la mayoría de los hostings no existe.
Menciónalo en el informe como pendiente; la opción es
`sg_security_notification_emails` y guarda un array de direcciones. No la cambies por tu
cuenta: el valor lo elige la persona.

---

## 4. Paso 3 · Fuentes del sistema

Primero, el ID de la entrada de estilos globales del tema activo:

```
db query "SELECT ID, post_name FROM {prefix}posts WHERE post_type = 'wp_global_styles'"
```

Después, `rest_api` con método `PUT` sobre `/wp/v2/global-styles/<ID>` y este cuerpo.
Va entero en una línea. Sustituye las claves `settings` y `styles` completas, así que si
la entrada ya tenía contenido, léela antes con un `GET` y funde lo que hubiera con esto:

```json
{"settings":{"typography":{"fontFamilies":{"custom":[{"fontFamily":"-apple-system, BlinkMacSystemFont, \"Segoe UI\", Roboto, \"Helvetica Neue\", Arial, sans-serif","name":"Sistema","slug":"system"},{"fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, \"Liberation Mono\", monospace","name":"Sistema monoespaciada","slug":"system-mono"}]}}},"styles":{"typography":{"fontFamily":"var(--wp--preset--font-family--system)"},"blocks":{"core/code":{"typography":{"fontFamily":"var(--wp--preset--font-family--system-mono)"}},"core/preformatted":{"typography":{"fontFamily":"var(--wp--preset--font-family--system-mono)"}},"core/verse":{"typography":{"fontFamily":"var(--wp--preset--font-family--system-mono)"}}}}}
```

El orden de cada pila importa: interfaz de macOS e iOS, luego Windows, luego Android y
escritorios Linux, y al final un genérico que siempre existe.

Purga y verifica en el HTML de la portada:

```
cache purge
get_page_html  →  url: la portada,  selector: head
```

La declaración del `body` tiene que decir
`font-family: var(--wp--preset--font-family--system)`. El bloque de estilos en línea de
la portada es grande (puede pasar de 50 KB), así que si no aparece en la primera
respuesta, repite la llamada con `offset` avanzando de 5.000 en 5.000 en lugar de darlo
por perdido. Las declaraciones `@font-face` del tema suelen estar cerca del final.

---

## 5. Medición

Siempre en dos tiempos: **calentar y luego medir.** Una página cacheada por primera vez
mide peor de lo que rinde.

1. `get_page_html` sobre la portada (calienta la caché).
2. `audit_page` sobre la misma URL, perfil móvil, **una llamada por categoría**:
   `performance`, `accessibility`, `best-practices`, `seo`. Agruparlas devuelve
   resultados inconsistentes.
3. En la primera auditoría después de cualquier cambio, pide `refresh` activo o
   PageSpeed devolverá el informe anterior en caché.

Anota las cuatro puntuaciones y LCP, CLS, TBT, FCP y Speed Index.

Interpretación honrada: entre dos ejecuciones seguidas, sin tocar nada, la puntuación
de rendimiento oscila hasta 15 puntos y el Speed Index es la métrica más ruidosa de las
cinco. Un 99 que pasa a 100 no demuestra nada por sí solo; lo que sí demuestra algo es
una petición menos y unos kilobytes menos, y eso lo dan las herramientas externas.
