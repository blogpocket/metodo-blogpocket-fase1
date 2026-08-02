# Incidencias conocidas

Todas están vistas en un sitio real durante la construcción del método. Cuando una de
estas aparezca, no investigues desde cero: el diagnóstico ya está hecho, y en casi todas
la pista superficial lleva al sitio equivocado.

## 401 en `/wp-json/` nada más empezar

**Síntoma.** `site_info` u otra llamada devuelve 401 antes de que WordPress llegue a
ejecutarse. El diagnóstico automático suele culpar a una protección del servidor.

**Causa real.** Credenciales guardadas caducadas en WPVibe.

**Qué hacer.** Eliminar el sitio de la cuenta de WPVibe y volver a conectarlo. El enlace
de autorización dura una hora. Es más rápido que investigar el servidor, y no hace falta
tocar nada del hosting.

## `rest_no_route` con el plugin activo y la ruta listada

**Síntoma.** El índice REST lista la ruta, el plugin está activo, y llamar a la ruta
devuelve 404.

**Causa real.** LiteSpeed guardó el 404 de esa ruta antes de que el plugin la
registrara, y lo sigue sirviendo. La cabecera `x-litespeed-cache: hit` sobre
`/wp-json/` lo delata, aunque `get_page_html` no devuelve cabeceras y no sirve para
verlo.

**Qué hacer.** `cache purge` y asegurarse de que `/wp-json/` está en las URI excluidas.
Es exactamente lo que previene el paso 1, así que después de ejecutarlo no debería
volver a pasar.

## 403 al tocar archivos del tema

**Síntoma.** `create_draft_theme` y todas las herramientas de archivos devuelven 403 con
un mensaje que habla de credenciales y sugiere reconectar el sitio.

**Causa real.** La protección «Desactivar el editor de temas y plugins» de Security
Optimizer retira la capacidad `edit_themes`, incluso al administrador, igual que haría
la constante `DISALLOW_FILE_EDIT`. No está en `wp-config.php`: la pone el plugin al
arrancar, así que `config get --type=constant` no la ve.

**Qué hacer.** Nada. Reconectar el sitio no arregla nada porque no hay nada roto. El
paso 3 escribe en los estilos globales precisamente por esto. Si algún día hiciera falta
editar un archivo del tema, la única vía es apagar la protección a mano, hacer el cambio
y volver a encenderla — y eso lo decide la persona, no la skill.

## El endpoint REST de Security Optimizer corrompe su propia opción

**Síntoma.** Una llamada a `/wp-json/sg-security/v1/...` devuelve un error fatal de PHP
en `Message_Service.php`, y a partir de ahí la opción que iba a modificar contiene un
objeto de respuesta del plugin en lugar de su valor.

**Qué hacer.** No usar esos endpoints. Los ajustes de un plugin son opciones de
WordPress y se escriben como tales. Para reparar la opción, borrarla y volverla a crear
con `option add`: `option update` se negará, porque no sobrescribe un valor serializado
con un escalar. El borrado pide aprobación en el navegador.

## `option update` rechaza el valor

**Síntoma.** La orden no da error claro pero el valor no cambia, o WPVibe avisa de que
va a escribir un escalar encima de algo guardado como objeto.

**Qué hacer.** `option delete` seguido de `option add`. El borrado pasa por la pantalla
de aprobación.

## Una orden sobre un plugin no surte efecto

**Síntoma.** `plugin deactivate <slug>` devuelve algo parecido a un éxito y el plugin
sigue activo.

**Qué hacer.** Usar la ruta completa del archivo: `loginizer/loginizer.php`, no
`loginizer`. Ante cualquier operación de plugins que no haga nada, la ruta completa es
lo primero que probar.

## «Ocultar la versión de WordPress» no oculta nada

**Síntoma.** La opción `sg_security_wp_remove_version` vale 1, la caché está purgada, y
la portada sigue emitiendo `<meta name="generator" content="WordPress ...">`.

**Causa probable.** Una respuesta cacheada anterior a la activación del plugin, o un
desajuste entre la versión de WordPress del sitio y la compatibilidad declarada del
plugin. Se vio una vez con WordPress 7.1-beta4 y un plugin declarado compatible hasta la
7.0.2, y en una ejecución posterior sobre el mismo sitio y las mismas versiones la
protección sí funcionó, así que la primera hipótesis es la más probable.

**Qué hacer.** Purgar la caché y volver a mirar el HTML antes de dar nada por sentado. Si
persiste, anotarlo en el informe como protección que no surte efecto y sugerir revisarlo
cuando el plugin se actualice. Lo que no vale es fiarse de la casilla del panel en
ninguno de los dos sentidos.

## XML-RPC devuelve 403 y no se sabe de quién es el mérito

Algunos hostings, GreenGeeks entre ellos, cortan esa ruta por su cuenta. La comprobación
da 403 igual con el plugin y sin él, así que no se puede atribuir. Si se menciona,
mencionarlo así.

## La instalación de un plugin no hace nada

Las instalaciones son de dos tiempos por diseño: la primera llamada informa de la
versión que se instalaría, y hay que repetirla confirmando la escritura. Si el plugin no
aparece, es que faltó la segunda llamada.

## Actualizar el núcleo o `wp-config.php` devuelve 403

Está bloqueado por diseño en WPVibe y no se puede negociar desde el chat. Esas dos cosas
las hace la persona desde el escritorio de WordPress. Ninguna de las dos hace falta para
la fase 1.
