---
name: metodo-blogpocket-fase1
description: Ejecuta de principio a fin la fase 1 del Método Blogpocket sobre un sitio WordPress conectado por WPVibe — mide la línea base, instala y configura LiteSpeed Cache, sustituye Loginizer por Security Optimizer (SG Security), pone fuentes del sistema en el tema de bloques y cierra con una medición comparada. Usa esta skill siempre que alguien pida construir, montar, dejar listo u optimizar un WordPress «con el Método Blogpocket», «según el método de Blogpocket», «la fase 1 del método», un «sitio inicial optimizado» o un «WordPress optimizado desde cero», aunque no nombre el método entero y aunque solo suelte una URL. Dispara también con «optimiza mi WordPress al 100 %», «deja este sitio optimizado en rendimiento y seguridad» o «pon la caché y la seguridad en [URL]», siempre que el sitio esté conectado por WPVibe. No la uses en sitios que ya tienen contenido y diseño propio, que es trabajo de la fase 2.
---

# Método Blogpocket · Fase 1

Deja un WordPress recién instalado en el estado que el Método Blogpocket llama «fase 1»: caché de página configurada, seguridad puesta, fuentes del sistema en lugar de las del tema, y una medición antes/después que lo demuestra.

Todo se ejecuta por WPVibe. La persona no abre el escritorio de WordPress salvo para aprobar en el navegador las operaciones destructivas, que WPVibe no deja aprobar desde el chat.

El método completo y el porqué de cada decisión están en `references/metodo.md`. Las llamadas exactas están en `references/comandos.md`. Los fallos conocidos, en `references/incidencias.md`. Lee `comandos.md` antes de ejecutar el paso 1: contiene los formatos exactos de `option update`, que no son evidentes y fallan en silencio si se escriben de otra manera.

## Cómo hablar durante la ejecución

Quien pide esto quiere el sitio hecho, no una clase. La regla es: **una línea por paso, y el informe al final.**

- No narres lo que vas a hacer antes de hacerlo, ni expliques por qué salvo que pregunten.
- No vuelques salidas de herramientas, tablas de opciones ni fragmentos de HTML.
- Entre paso y paso basta con algo del estilo de «Paso 1 hecho: LiteSpeed Cache instalado y configurado. 100/100/100/91.»
- El único bloque largo de la conversación es el informe de cierre.
- Si algo falla, dilo en dos frases: qué ha fallado y qué hace falta para seguir.

## Antes de tocar nada

1. **La URL.** Si no la han dado, pídela y nada más. Con la URL, `site_info` sobre ese sitio.
2. **Inventario.** Anota tema activo, plugins instalados, versión de WordPress y PHP, y cuánto contenido hay. Con `db query` cuenta entradas y páginas propias (ver `comandos.md`).
3. **Revisa las condiciones de parada** de la sección siguiente.
4. **Un solo mensaje de confirmación** con la plantilla de abajo, y espera el visto bueno. Es la única vez que preguntas algo antes del final.

### Plantilla de confirmación

```
[dominio] · listo para la fase 1.

He encontrado: [tema] · [n] plugins ([lista]) · WordPress [x] · PHP [y] · [contenido].

Voy a: medir la línea base, instalar LiteSpeed Cache, [desinstalar Loginizer e]
instalar Security Optimizer, cambiar a fuentes del sistema y volver a medir.

Dos avisos: el hosting tiene que ser LiteSpeed (GreenGeeks lo es) para que la caché
de página funcione, y a mitad de camino te saldrá una pantalla de aprobación en el
navegador que tienes que confirmar tú.

¿Adelante?
```

### Condiciones de parada

Esta skill desinstala un plugin, borra opciones y reescribe los estilos globales. Sobre un sitio en marcha eso se nota. No sigas sin permiso explícito si:

- **Hay contenido propio.** Más allá de lo que trae la instalación (`Hello world!`, `Sample Page`, la página del hosting, el borrador de política de privacidad), el sitio está en uso. Dilo y pregunta.
- **Hay plugins que no esperabas.** Lo previsto es WPVibe y, si el hosting lo trae, Loginizer. Cualquier otro —incluidos los que algunos hostings o modos de trabajo añaden por su cuenta— lístalo y pregunta si se queda antes de seguir. No des ninguno por bueno solo porque parezca inofensivo.
- **El tema activo no es de bloques.** Sin entrada `wp_global_styles` el paso 3 no tiene dónde escribir. Hazlo sin el paso 3 o para, según lo que digan.
- **Ya hay un plugin de caché distinto.** Dos cachés se pelean. Pregunta cuál se queda.
- **`site_info` falla.** Casi siempre son credenciales caducadas, no el servidor. Ver `incidencias.md`.

## Paso 0 · Línea base

Sin esto el resto no significa nada: es la cifra contra la que se compara al final.

1. Calienta la caché pidiendo la portada una vez.
2. Lanza la auditoría de Lighthouse en móvil, **una llamada por categoría** (agruparlas da resultados inconsistentes), con `refresh` activo en la primera.
3. Anota las cuatro puntuaciones y los cinco Core Web Vitals (LCP, CLS, TBT, FCP, Speed Index). Si la salida trae peso total y número de peticiones, anótalos también: son la cifra que más se mueve al final.

Di solo: «Línea base: [p]/[a]/[bp]/[seo].»

## Paso 1 · LiteSpeed Cache

Instala `litespeed-cache` activándolo. La instalación es de dos tiempos: la primera llamada informa de la versión y hay que repetirla confirmando.

Los valores de fábrica de la 7.8 ya son razonables, así que solo cambian tres ajustes:

| Ajuste | Opción | Valor |
|---|---|---|
| No cachear URI | `litespeed.conf.cache-exc` | `/wp-json/` |
| Cachear la REST API | `litespeed.conf.cache-rest` | desactivado |
| Caché del navegador | `litespeed.conf.cache-browser` | activado |

Los dos primeros existen para que la caché no fosilice un 404 de una ruta REST que todavía no está registrada, que es la avería más desconcertante de este montaje. El tercero no mueve Lighthouse, pero sí la visita recurrente.

**No toques la pestaña de optimización de página** (minificar, combinar, aplazar, CSS crítico, quitar emojis). Cada interruptor es un cambio que pediría su propia medición, y en un sitio de cinco peticiones no hay nada que combinar. Eso es fase 2.

Verifica las tres opciones con una consulta a la base de datos, purga, calienta y mide.

## Paso 2 · Security Optimizer

### Si hay Loginizer

Desactívalo y desinstálalo. **Nómbralo por su archivo, no por su carpeta**: `loginizer/loginizer.php`. Con el slug solo, la orden falla sin decir nada.

La desinstalación pide aprobación en el navegador. Avisa en una línea («te saldrá una pantalla de aprobación») y espera.

Después, comprueba los restos en los cuatro sitios donde suelen quedarse: opciones, tablas propias, metadatos de usuario y tareas programadas. Lo habitual es encontrar una opción huérfana (`loginizer_pro_less_than_201`); bórrala, que también pide aprobación. Esta comprobación de cuatro puntos vale para cualquier plugin que se desinstale.

### Instalación

Instala `sg-security` activándolo (el plugin que se conocía como SG Security se llama hoy Security Optimizer; es el mismo). Trae **siete protecciones ya encendidas** y la configuración básica de la fase 1 es exactamente esa: no hay nada que cambiar. Verifica que las siete opciones están escritas (lista en `comandos.md`).

Tres cosas se quedan apagadas a propósito: la URL de acceso personalizada y la verificación en dos pasos, que son decisiones de la persona y van en fase 2, y la desactivación de los feeds RSS, que un blog necesita.

### Comprobación a mano

Un panel que dice «activo» no prueba nada. Mira el HTML de la portada y busca la etiqueta `generator`. Si sigue mostrando la versión de WordPress, «Ocultar la versión» no está surtiendo efecto: pasa en versiones de WordPress más nuevas que la compatibilidad declarada del plugin. Anótalo para el informe en lugar de callarlo.

Purga, calienta y mide. Lo esperable es que las cinco métricas no se muevan ni un punto: este plugin no encola nada en el frontend. Medir igualmente es lo que permite afirmarlo.

## Paso 3 · Fuentes del sistema

El tema trae fuentes propias (en Twenty Twenty-Five, Manrope y Fira Code) y esa descarga suele ser **más de la mitad del peso de la portada**. Se sustituyen por pilas del sistema, que no descargan nada.

Editar el `theme.json` del tema no vale, por dos razones: el cambio desaparece en la primera actualización del tema, y además la protección «Desactivar el editor de temas y plugins» del paso 2 retira la capacidad `edit_themes` y devuelve 403 a cualquier herramienta de archivos. El mensaje de error habla de credenciales y de reconectar el sitio; es una pista falsa.

El sitio correcto son los **estilos globales**: una entrada de tipo `wp_global_styles`, una por tema, que sobrevive a las actualizaciones. Localiza su ID y escríbela por REST con el cuerpo completo de `comandos.md` — el `PUT` sustituye las claves `settings` y `styles` enteras, así que hay que enviarlo todo, no solo lo que cambia.

Después, purga y **verifica en el HTML de la portada** que la declaración del `body` ya apunta a `var(--wp--preset--font-family--system)` y no a la fuente del tema. Ese es el único punto de este paso que se puede afirmar con certeza; Lighthouse apenas lo notará.

Los `@font-face` del tema siguen imprimiéndose (medio kilobyte de CSS en línea) porque los registra el tema, pero ninguna regla los invoca, así que no se descarga ningún archivo. Es un límite conocido, no un fallo.

Calienta y mide.

## Cierre · El informe

Es lo único largo que se escribe. Rellena esta plantilla **entera**, sin quitar ni añadir apartados:

```
**[dominio] · fase 1 del Método Blogpocket completada.**

| Categoría | Antes | Después |
|---|---|---|
| Rendimiento | | |
| Accesibilidad | | |
| Buenas prácticas | | |
| SEO | | |

| Métrica | Antes | Después |
|---|---|---|
| LCP | | |
| CLS | | |
| TBT | | |
| FCP | | |
| Speed Index | | |

**El sitio, ahora:** [tema], [plugins], caché activa con /wp-json/ excluido,
[n] protecciones de seguridad, fuentes del sistema.

**Pendiente para la fase 2:**
1. La meta description. Es lo que deja el SEO por debajo de 100 y necesita un
   plugin de SEO, que no cabe en la regla de los dos plugins.
2. La verificación en dos pasos. Es la medida que más sube la seguridad real
   del acceso, y se activa cuando el sitio tenga dueño y rutina.
3. La URL de acceso personalizada. Decisión de proyecto, no de método.
4. La dirección de las notificaciones de Security Optimizer, que apunta a una
   cuenta que en la mayoría de los hostings no existe. El valor lo eliges tú.
[5. Cualquier protección que no haya surtido efecto, si la hay.]

Las dos mediciones externas hay que lanzarlas a mano: digitalbeacon.co y
gtmetrix.com. Si me pegas el resultado, lo comparo.
```

Los cuatro puntos numerados van **siempre**, aunque parezcan obvios y aunque el sitio no dé señales de necesitarlos: son las decisiones que la fase 1 aplaza a conciencia, y el informe es donde el lector se entera de que existen. No los sustituyas por otras observaciones. Si tienes algo más que contar —un plugin ajeno al método que se queda, contenido de la instalación sin tocar—, va después de la plantilla, no dentro.

Y no nombres productos que el método no elige. La meta description «necesita un plugin de SEO», no un plugin concreto: quién lo instale lo decidirá en la fase 2. Lo mismo con cualquier otra herramienta que no esté en `references/metodo.md`.

## Si algo se tuerce

Consulta `references/incidencias.md` antes de improvisar. Están documentados el 401 de la REST API, el 403 de las herramientas de tema, el endpoint de Security Optimizer que corrompe su propia opción, y el `option update` que se niega a escribir sobre un valor serializado.

Dos reglas que evitan casi todo:

- **Los ajustes de un plugin se escriben como opciones de WordPress**, nunca por los endpoints REST que el plugin publique para su propio panel. Esos endpoints esperan cabeceras y nonces que una llamada externa no reproduce, y uno de ellos llega a guardar su propio objeto de error dentro de la opción que iba a cambiar.
- **Actualizar el núcleo y tocar `wp-config.php` están bloqueados** por diseño. Si hiciera falta, lo hace la persona desde el escritorio.
