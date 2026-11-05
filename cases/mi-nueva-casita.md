# Mi Nueva Casita — panel de búsqueda de arriendo

**Un panel privado que revisa las inmobiliarias cada 12 horas, filtra con criterios
explícitos y explica cada decisión.**

[Código](https://github.com/rubendariogomeza/mi-nueva-casita)

**Rol:** producto, diseño y desarrollo.
**Stack:** Next.js 15 (App Router), React 19, TypeScript, `node:sqlite`, sin librerías de UI.
**Estado:** funcionando en local. Fase 2 (acceso desde el celular) pendiente.

---

## El problema

Buscar arriendo en Medellín significa revisar cuatro o cinco portales, comparar precios
que no son comparables, adivinar la administración —que casi nunca publican— y volver a
empezar cada semana porque las ofertas buenas se van rápido.

El dolor real no es "no hay dónde buscar". Es **no revisar a tiempo y no poder comparar**.

## Lo que hace

1. **Rastrea** tres inmobiliarias locales con sitio abierto, cada 12 horas.
2. **Filtra** cada oferta con los criterios del usuario y la clasifica en tres estados.
3. **Puntúa** de 0 a 100 con motivos visibles.
4. **Registra** las decisiones del usuario: descartar, recuperar, contactar.
5. **Avisa** con un resumen por correo o un archivo HTML si no hay correo configurado.

## La decisión central: tres estados, no dos

La mayoría de los filtros son binarios: cumple o no cumple. Ese diseño castiga los datos
incompletos, y en inmuebles los datos casi siempre están incompletos.

Por eso el filtro tiene tres salidas:

| Estado | Cuándo | Qué hace el sistema |
| --- | --- | --- |
| **Cumple** | Zona correcta, área suficiente, precio total dentro del techo | La muestra primero |
| **Por confirmar** | Falta un dato (área, precio, barrio, administración) | No la descarta: la deja para que el usuario decida |
| **Descartada** | Falla un criterio duro | La guarda con el motivo y permite recuperarla |

Dos reglas derivadas, y son las que hacen que el panel sea confiable:

- **La administración se estima por zona y se marca como estimada.** Ninguna fuente la
  publica. Fingir certeza sería peor que estimar con la etiqueta visible.
- **Balcón y espacio de trabajo nunca descartan.** Las fuentes casi no los mencionan;
  descartar por ausencia de dato eliminaría ofertas que sí cumplen.

## El puntaje

0 a 100, con los motivos a la vista: precio por m² frente al mercado de la zona (40),
administración sobre el canon (20), frescura (15), tamaño (10), amenidades (10) y señales
del texto (5). Un puntaje sin explicación es una caja negra; con motivos, es una
herramienta para decidir.

## La restricción legal, tratada como requisito de producto

Los portales grandes —Fincaraiz, Properati, Ciencuadras, Metrocuadrado— prohíben el acceso
automatizado en sus términos. La app **no los rastrea**. Solo usa inmobiliarias con sitio
abierto y `robots.txt` que lo permite, con dos segundos entre peticiones.

Facebook se evaluó y se descartó con un documento propio: el riesgo de suplantación y de
datos personales no justificaba el beneficio.

Esto no es un detalle técnico: es la diferencia entre una herramienta que se puede usar y
una que expone al usuario.

## Decisiones de ingeniería

- **Local primero.** La base es un archivo SQLite. Sin cuenta, sin nube, sin costo. El
  despliegue remoto queda para la fase 2, cuando el valor esté probado.
- **Sin librerías de UI.** Next.js y React, nada más. El panel es pequeño y el costo de
  mantenimiento importa más que la comodidad de un framework de componentes.
- **Login con cookie firmada por HMAC.** Una sola dependencia conceptual, sin proveedor de
  identidad para un panel de dos usuarios. Sin `APP_SECRET` ni usuarios configurados el
  panel no arranca: no hay credenciales por defecto.
- **Tarea programada de Windows** cada 12 horas, con registro en `data/ingest.log` y una
  pantalla de **Revisiones** que muestra si una fuente falló.
- **Pruebas sobre la lógica pura** (`npm run test:logic`): filtros, puntaje y motivos.
  Typecheck completo antes de cada cambio.

## Lo que este proyecto demuestra

No es el código. Es el proceso que está documentado en `docs/`:

`00-descubrimiento` · `01-brief` · `02-requisitos` · `03-backlog` · `04-riesgos` ·
`05-decisiones` · `06-estado` · `07-arquitectura` · `investigacion/`

Un descubrimiento con rondas de preguntas, un brief, requisitos, riesgos con probabilidad e
impacto, decisiones fechadas y un registro de estado. Es la misma disciplina que aplico a
un proyecto de software de un cliente, aplicada a un proyecto de una sola persona.

## Límites conocidos

- La primera revisión tarda unos 15 minutos por diseño (dos segundos entre peticiones).
- Si una inmobiliaria cambia su sitio, la extracción falla y hay que ajustar el adaptador;
  el panel lo muestra en Revisiones.
- Corre en el computador del usuario. Verlo desde el celular exige la fase 2.

## Siguiente paso

Convertir el panel en una aplicación instalable, con notificaciones y acceso remoto, sin
perder la propiedad de los datos: la base sigue siendo del usuario.
