# El Método Blogpocket, en corto

Lee esto si preguntan por qué se hace algo, o si hay que justificar una decisión. Durante
la ejecución normal no hace falta: la skill construye, no explica.

Método de Antonio Cambronero (Blogpocket) para construir un sitio WordPress optimizado
sobre cuatro pilares: **rendimiento, seguridad, SEO y legalidad**. El propietario del
sitio no escribe código en ningún momento.

## La idea

Un sitio WordPress se degrada por acumulación. Un plugin porque resuelve un problema, un
tema que trae sus propias fuentes, un constructor visual, y a los pocos meses la página
tarda cuatro segundos. Arreglarlo entonces cuesta más que haberlo evitado.

El método invierte el orden: primero un sitio mínimo y limpio, se comprueba que rinde
bien, y solo entonces se añade lo que el sitio necesite. Cada añadido se mide en el
momento en que se hace.

## Las dos fases

**Fase 1 — sitio inicial optimizado.** Es lo que ejecuta esta skill:

1. WordPress en un hosting verde.
2. Tema base por defecto, del estilo de Twenty Twenty-Five.
3. Dos plugins, y solo dos: LiteSpeed Cache (en hostings con servidor LiteSpeed) y
   Security Optimizer, antes SG Security.
4. Configuración básica de ambos.
5. El tema no se toca, con una excepción: fuentes del sistema y fuera cualquier otra.

Al cerrar la fase 1 el sitio está optimizado en rendimiento y seguridad. Todavía no
tiene contenido, ni diseño propio, ni páginas legales.

**Fase 2 — personalización.** Páginas, contenido, formularios, textos legales, plugins
de SEO. Depende del proyecto y no está cubierta por esta skill.

## La regla que sostiene el método

**En cada paso se mide.** Creas una página, mides. Instalas un plugin, mides. Cambias una
fuente, mides.

Medir después de cada cambio, y no al final, es lo que permite identificar al culpable.
Si el rendimiento cae de 99 a 74 justo después de activar un plugin, ya sabes qué plugin
lo hizo. Si esperas a tener quince cosas instaladas, el diagnóstico se convierte en una
investigación.

De ahí que la skill mida cuatro veces —línea base y una por paso— aunque el resultado sea
idéntico tres de esas veces. Un plugin de seguridad que cargara scripts en cada página lo
habría enseñado ahí, y la única manera de saberlo era mirar.

## Loginizer

Muchos hostings, GreenGeeks entre ellos, lo instalan junto con WordPress. No es una
elección del método: viene puesto. Cumple una función mientras está, porque limita los
intentos de acceso desde el minuto uno.

Ese es todo su papel: un puente entre la instalación del hosting y la fase 1. Cuando
entra Security Optimizer, Loginizer se desinstala. Dos plugins vigilando el mismo
formulario de acceso no protegen el doble; se estorban, duplican bloqueos y complican el
diagnóstico cuando alguien no puede entrar.

Si el hosting no lo trae, no se instala. No se pierde nada.

## Por qué el hosting condiciona el paso 1

LiteSpeed Cache solo hace caché de página cuando el servidor es LiteSpeed u
OpenLiteSpeed. GreenGeeks lo es. En Apache o Nginx sin LiteSpeed el plugin se instala
pero no cachea, y hace falta otro plugin con otra configuración, que el método no
describe. De ahí la pregunta antes de empezar.

## Por qué tres herramientas de medición y no una

- **Lighthouse** (móvil, PageSpeed Insights) mide la experiencia de carga. Es la
  medición de cada paso y la única que la skill puede lanzar sola.
- **Digital Beacon** da emisiones de CO2 por visita y el desglose de peso.
- **GTmetrix** da peso total y número de peticiones.

La razón de arrastrar las tres se vio al cerrar la fase 1 en el sitio de referencia: el
rendimiento de Lighthouse pasó de 99 a 100, un punto que cabe entero dentro del ruido, y
con esa única señal la conclusión razonable habría sido que la fase no sirvió de nada.
Mientras tanto la página había perdido dos tercios de su peso y el 64 % de sus
emisiones. Lighthouse mide una experiencia de carga, no un consumo.

Las dos externas hay que lanzarlas a mano. La skill las ofrece al final y compara si se
las pegan.

## Cifras de referencia del sitio donde se construyó el método

Instalación limpia de WordPress con Twenty Twenty-Five, en GreenGeeks:

| | Estado cero | Fase 1 cerrada |
|---|---|---|
| Lighthouse móvil | 99 / 100 / 100 / 91 | 100 / 100 / 100 / 91 |
| LCP · CLS · TBT · FCP · SI | 1,3 s · 0 · 0 ms · 0,9 s · 3,1 s | 0,8 s · 0 · 0 ms · 0,8 s · 0,9 s |
| CO2 por visita | 0,025 g | 0,009 g |
| Peso comprimido | 80,6 KB | 28,4 KB |
| Peticiones | 5 | 4 |

Los 52 KB que desaparecen son un solo archivo: la fuente variable que traía el tema. Una
instalación limpia de WordPress con su tema por defecto es, en peso, mayoritariamente
una tipografía.

Los dos plugins del método no añadieron ni una petición ni un byte al frontend.

Sirven de orientación, no de objetivo: otro hosting, otro tema u otra versión de
WordPress dan cifras distintas.

## Lo que la fase 1 deja pendiente a propósito

- **La meta description.** Es lo único que separa el SEO de 91 de 100, y requiere un
  plugin de SEO, que no cabe en la regla de los dos plugins. Es el primer añadido de la
  fase 2, y el primero que hay que medir.
- **La verificación en dos pasos.** Es la medida que más sube la seguridad real del
  acceso, y la recomendación es activarla. Obliga a configurar una aplicación de
  autenticación en el siguiente acceso, y eso es tarea de la persona.
- **La URL de acceso personalizada.** Evita el ruido de los bots, no protege de quien ya
  conozca la ruta, y añade una forma nueva de quedarse fuera del propio sitio. Decisión
  de proyecto.
- **El grosor del texto.** Los temas suelen pedir un `font-weight` que las fuentes del
  sistema no siempre tienen, y el navegador lo resuelve con el peso más cercano. Es
  diseño, y el diseño es fase 2.
