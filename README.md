# Método Blogpocket · Fase 1 (skill para Claude)

Construye la **fase 1 del Método Blogpocket** sobre un WordPress recién instalado, desde
una conversación con Claude: mide la línea base, instala y configura LiteSpeed Cache,
sustituye Loginizer por Security Optimizer, pone fuentes del sistema en lugar de las del
tema y cierra con una medición comparada.

Una frase basta:

> Construye en midominio.com un sitio web inicial optimizado con el Método Blogpocket.

## Antes de usarla

- WordPress instalado, con un tema de bloques activo (del estilo de Twenty Twenty-Five).
- Hosting con servidor **LiteSpeed** — GreenGeeks lo es — y HTTPS activo.
- El plugin [WPVibe](https://wordpress.org/plugins/vibe-ai/) instalado y el sitio
  conectado a tu cuenta de Claude.
- El sitio **sin contenido propio**. La skill desinstala un plugin y reescribe los
  estilos globales; sobre un sitio en marcha eso se nota, y por eso se para y pregunta
  si encuentra contenido publicado.

Loginizer es opcional. Si el hosting lo trae puesto, la skill lo quita.

## Instalación

Descarga `metodo-blogpocket-fase1.skill`, adjúntalo a una conversación en Claude y pulsa
**Save skill** en la tarjeta del archivo.

## Documentación

`guia-skill-fase1.pdf` cubre la instalación, el uso, lo que la skill pide por el camino,
sus límites conocidos y un plan de pruebas de cinco casos para comprobarla antes de
ponerla en manos de otras personas.

El método completo, con la fase 1 documentada paso a paso y medida sobre un sitio real,
está en el manual del Método Blogpocket.

## Contenido

```
metodo-blogpocket-fase1/
├── SKILL.md                    el flujo entero
└── references/
    ├── comandos.md             las llamadas exactas
    ├── incidencias.md          los fallos conocidos
    └── metodo.md               el método en corto
```

## Qué no hace

No crea el sitio, no hace la fase 2 (contenido, diseño, textos legales, plugin de SEO),
no toca la meta description, no actualiza el núcleo de WordPress y no lanza Digital
Beacon ni GTmetrix, que son herramientas externas.

---

Antonio Cambronero · [Blogpocket](https://www.blogpocket.com)
