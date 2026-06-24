# Proyecto N°2 — Diseño de Software
## Bounded Contexts de YouTube · Semestre Otoño 2026

---

## Descripción general

Este repositorio divide el negocio de YouTube en 6 bounded contexts independientes,
cada uno con su propio modelo de datos, reglas de negocio y especificación OpenAPI.
El diseño sigue los principios de Domain-Driven Design (DDD): cada contexto conoce el
mismo concepto de negocio (un video, un canal, un usuario) de una forma distinta,
según la responsabilidad que le corresponde.

---

## Los 6 bounded contexts

| # | Contexto | Responsabilidad central |
|---|---|---|
| 1 | Publicación y distribución de contenido | Ciclo de vida técnico del video: subida, procesamiento, reproducción y live |
| 2 | Catálogo editorial y derechos | Qué contenido existe públicamente, cómo se presenta y bajo qué reglas puede mostrarse |
| 3 | Descubrimiento y personalización | Cómo el usuario encuentra contenido: búsqueda, feeds, recomendaciones y ranking |
| 4 | Audiencia, comunidad y engagement | Relación social entre usuarios, creadores y contenido |
| 5 | Monetización del ecosistema creador | Cómo los creadores ganan dinero y cómo se registran, calculan y pagan esos ingresos |
| 6 | Publicidad y marketplace de anunciantes | Negocio B2B: campañas, inventario, targeting, entrega de anuncios e ingresos para la plataforma |

---

## Estructura del repositorio

```
contexts/
├── 01-publicacion-distribucion/
│   ├── openapi.yaml
│   ├── openapi.json
│   ├── diagrama-clases.md
│   ├── diagrama-secuencia.md
│   ├── diagrama-clases.drawio
│   ├── diagrama-secuencia.drawio
│   └── README.md
├── 02-catalogo-editorial-derechos/
│   └── ...
├── 03-descubrimiento-personalizacion/
│   └── ...
├── 04-audiencia-comunidad-engagement/
│   └── ...
├── 05-monetizacion-ecosistema-creador/
│   └── ...
└── 06-publicidad-marketplace-anunciantes/
    └── ...
```

---

## Dependencias entre contextos

Según la sección 4 del enunciado, los contextos se integran de la siguiente forma:

```
Publicación ──AssetReadyForPublishing──► Catálogo
Catálogo ──ContentPublished/Unpublished──► Descubrimiento, Monetización, Publicidad
Audiencia ──señales de engagement──► Descubrimiento
Publicidad ──AdRevenueGenerated──► Monetización
Publicación ──provee reproducción──► Audiencia registra interacciones
```

Todos los eventos de integración son **asíncronos**: el contexto que emite no espera
respuesta del que recibe. Las consultas síncronas entre contextos se limitan a
validaciones de existencia o visibilidad, siguiendo el RNF-4 del enunciado.

---

## Nota de diseño: nombres de identificadores entre contextos

A lo largo de los 6 specs se observa que el mismo identificador de un ítem de
contenido aparece con nombres distintos según el contexto:

| Contexto | Nombre del campo | Perspectiva que representa |
|---|---|---|
| Catálogo (C2) | `contentId` | El ítem de catálogo editorial que el creador gestiona |
| Descubrimiento (C3) | `catalogItemId` | Un candidato de ranking con señales de interés |
| Audiencia (C4) | `catalogItemId` | El contenido con el que el usuario interactúa socialmente |
| Monetización (C5) | `videoId` | Una unidad monetizable que genera ingresos |
| Publicidad (C6) | `videoId` | Un slot publicitario dentro del inventario |

Esta variación es **intencional y coherente con DDD**. El RNF-2 del enunciado
establece que los contextos pueden compartir identificadores pero no el mismo modelo
de datos. El nombre del campo en cada contexto refleja el **lenguaje ubicuo** propio
de ese dominio:

- Para el equipo de Catálogo, ese identificador es un "contenido editorial".
- Para el equipo de Descubrimiento y Audiencia, es un "ítem de catálogo" al que
  referencian sin duplicar su metadata.
- Para los equipos de Monetización y Publicidad, que operan en lógica de negocio
  de ingresos, el mismo objeto es naturalmente un "video" en el sentido comercial
  del término.

En todos los casos, el **valor del identificador es el mismo UUID** generado por
Catálogo al publicar el ítem. La diferencia es solo de nombre, no de semántica de
negocio ni de tipo de dato. Esta es una consecuencia directa del principio de
lenguaje ubicuo de DDD: la misma realidad recibe nombres distintos según el contexto
que la describe (sección 2.1 del enunciado).

---

## Escenario integrador (sección 13 del enunciado)

*"Un espectador ve un video recomendado, mira un anuncio, deja un like y el creador genera ingresos."*

| Paso | Responsabilidad | Endpoint |
|---|---|---|
| 1. Obtener recomendación | Descubrimiento (C3) | `GET /feeds/home` |
| 2. Validar que el contenido es visible | Catálogo (C2) | `GET /content/{contentId}/visibility` |
| 3. Elegir anuncio | Publicidad (C6) | `POST /ad-decisions` |
| 4. Reproducir video | Publicación (C1) | `GET /video-assets/{videoAssetId}/playback-info` |
| 5. Registrar like | Audiencia (C4) | `PUT /content/{catalogItemId}/reactions/me` |
| 6. Registrar watch time | Descubrimiento (C3) | `POST /signals/watch-time` |
| 7. Atribuir ingreso al creador | Monetización (C5) | `POST /integration/ad-revenue-confirmed` |

Cada paso es responsabilidad de un contexto distinto. Ningún servicio individual
resuelve más de un paso — eso confirma que los límites entre contextos son correctos.

---

## Equipo

| Integrante | Contextos a cargo |
|---|---|
| [Emmanuel Arancibia] | Publicación y distribución (C1) · Catálogo editorial y derechos (C2) |
| [Evan Morales] | Descubrimiento y personalización (C3) · Audiencia, comunidad y engagement (C4) |
| [Javier Orellana] | Monetización del ecosistema creador (C5) · Publicidad y marketplace de anunciantes (C6) |
