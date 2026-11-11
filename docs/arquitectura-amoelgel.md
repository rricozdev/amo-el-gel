# Amo el Gel — Arquitectura

Landing page + galería para una Nail Artist (BIAB · Builder Gel · Soft Gel · Pedicure) en Tel Aviv y Jerusalén. Monolito en un solo repo, sin backend, 4 idiomas (HE, EN, ES, RU).

## 1. Principios

- Alcance mínimo: una página, 5 secciones, sin base de datos ni CMS.
- Un componente = una responsabilidad (SRP). Sin abstracciones prematuras.
- Contenido separado de la UI (textos en `messages/`, datos en `content/`).
- RTL y multi-idioma desde el día 1, no como parche.
- Estética propia: esquinas rectas, líneas finas, tipografía grande, rojo como único acento. Evitar el "AI aesthetic" genérico (degradados morados, glassmorphism, tarjetas redondeadas por todos lados).

## 2. Stack

| Capa | Herramienta |
|---|---|
| Framework | Next.js (App Router, JSX) |
| Estilos | Tailwind CSS (propiedades lógicas para RTL) |
| Componentes | shadcn/ui (solo Button, Dialog, Sheet) |
| Scroll suave | Lenis |
| Animaciones | GSAP + ScrollTrigger |
| i18n | next-intl |
| Imágenes | `next/image` (archivos locales en `public/`) |
| Contacto | Enlace `wa.me` con mensaje prellenado por idioma |
| Deploy | Vercel |

## 3. Diseño (resumen)

**Paleta**

| Rol | Hex |
|---|---|
| Fondo | `#FAF5F0` |
| Rojo marca | `#C80A03` |
| Burdeos | `#5C0501` |
| Nude | `#E9D5CB` |
| Tinta | `#1F1416` |

**Fuentes** (todas con subsets según idioma)

| Idioma | Títulos | Texto |
|---|---|---|
| HE | Frank Ruhl Libre | Rubik |
| EN / ES / RU | Cormorant Garamond | Rubik |

**Estilo**: botones de esquinas casi cuadradas, galería masonry, lightbox, animaciones sutiles.

## 4. Estructura del repo

```
amoelgel/
├─ docs/
│  ├─ README.md              # índice de la documentación
│  ├─ architecture.md        # este documento
│  ├─ design-system.md       # paleta, fuentes, espaciados, botones
│  ├─ i18n.md                # idiomas, RTL, cómo agregar textos
│  ├─ content-guide.md       # cómo agregar fotos y editar textos
│  └─ deployment.md          # Vercel, dominio, variables de entorno
├─ messages/
│  ├─ he.json
│  ├─ en.json
│  ├─ es.json
│  └─ ru.json
├─ public/
│  └─ images/
│     ├─ logo/               # logo.svg / logo.webp
│     ├─ hero/
│     ├─ about/
│     └─ gallery/            # 01.webp, 02.webp, ...
├─ src/
│  ├─ app/
│  │  ├─ [locale]/
│  │  │  ├─ layout.jsx       # <html lang dir>, fuentes, providers
│  │  │  └─ page.jsx         # compone las secciones
│  │  ├─ globals.css         # tokens (colores, fuentes) + Tailwind
│  │  ├─ sitemap.js
│  │  └─ robots.js
│  ├─ components/
│  │  ├─ layout/
│  │  │  ├─ Navbar.jsx
│  │  │  ├─ LanguageSwitcher.jsx
│  │  │  ├─ Footer.jsx
│  │  │  └─ SmoothScroll.jsx # inicializa Lenis + sync con ScrollTrigger
│  │  ├─ sections/
│  │  │  ├─ Hero.jsx
│  │  │  ├─ Services.jsx
│  │  │  ├─ Gallery.jsx
│  │  │  ├─ About.jsx
│  │  │  └─ Contact.jsx
│  │  ├─ shared/
│  │  │  ├─ Reveal.jsx       # wrapper GSAP de entrada (respeta RTL)
│  │  │  ├─ Lightbox.jsx     # Dialog con navegación
│  │  │  ├─ WhatsAppButton.jsx
│  │  │  └─ SectionTitle.jsx
│  │  └─ ui/                 # generado por shadcn/ui
│  ├─ content/
│  │  ├─ site.js             # nombre, WhatsApp, Instagram, ciudades
│  │  ├─ gallery.js          # lista de imágenes (src, alt key, w, h)
│  │  └─ services.js         # ids de servicios (textos van en messages/)
│  ├─ i18n/
│  │  ├─ routing.js          # locales, defaultLocale
│  │  ├─ request.js          # carga de mensajes por locale
│  │  └─ navigation.js       # Link/redirect con locale
│  ├─ lib/
│  │  ├─ fonts.js            # next/font por idioma
│  │  ├─ whatsapp.js         # buildWhatsAppUrl(locale)
│  │  └─ utils.js            # cn() de shadcn
│  └─ proxy.js               # middleware de next-intl (nombre según versión de Next)
├─ components.json           # config shadcn/ui
├─ jsconfig.json             # alias @/*
├─ next.config.mjs
├─ package.json
└─ README.md
```

## 5. Páginas y secciones

Una sola ruta: `/[locale]` (`/he`, `/en`, `/es`, `/ru`).

| # | Sección | Contenido |
|---|---|---|
| 1 | Hero | Logo, frase principal, botón WhatsApp, foto macro |
| 2 | Services | BIAB, Builder Gel, Soft Gel, Pedicure (nombre + descripción corta) |
| 3 | Gallery | Masonry + lightbox |
| 4 | About | Quién es, idiomas que atiende, Tel Aviv y Jerusalén |
| 5 | Contact | Botón WhatsApp, enlace a Instagram, ciudades |

Navbar con anclas a cada sección + selector de idioma + botón de reserva.

## 6. Internacionalización

- Locales: `he` (por defecto), `en`, `es`, `ru`. Detección por `Accept-Language` en el proxy/middleware de next-intl.
- `layout.jsx` define `<html lang={locale} dir={locale === "he" ? "rtl" : "ltr"}>`.
- Todos los textos en `messages/<locale>.json`, con las mismas claves en los 4 archivos.
- Datos no traducibles (WhatsApp, Instagram, rutas de imágenes) en `content/`.
- Metadata (`title`, `description`, `og`) y `hreflang` generados por locale.
- Los textos de HE y RU deben ser revisados por un hablante nativo antes de enviar la demo.

### RTL

- Usar solo propiedades lógicas de Tailwind: `ms-`, `me-`, `ps-`, `pe-`, `text-start`, `border-s`, `start-0`.
- Iconos direccionales (flechas del lightbox) se voltean con `rtl:-scale-x-100`.
- Sin secciones de scroll horizontal (conflictivas con Lenis + RTL).
- Animaciones con desplazamiento en X usan `--dir` (`1` en LTR, `-1` en RTL).

## 7. Fuentes

Cargadas con `next/font/google` en `lib/fonts.js`, expuestas como variables CSS (`--font-display`, `--font-body`) y elegidas por idioma en el layout.

| Fuente | Subsets |
|---|---|
| Cormorant Garamond | latin, cyrillic |
| Frank Ruhl Libre | latin, hebrew |
| Rubik | latin, cyrillic, hebrew |

Verificar glifos y pesos antes de fijar.

## 8. Animaciones y scroll

- `SmoothScroll.jsx` (client): inicia Lenis y lo conecta a `ScrollTrigger` con `lenis.on("scroll", ScrollTrigger.update)` y `gsap.ticker`.
- `Reveal.jsx` (client): fade + translate al entrar en viewport; lee `--dir` para RTL.
- Parallax leve solo en Hero y fotos de About.
- Respetar `prefers-reduced-motion`: desactivar Lenis y reveals.
- Limpiar triggers con `gsap.context()` / `revert()` en el cleanup.

## 9. Galería

- `content/gallery.js` exporta un arreglo `{ id, src, width, height, altKey }`.
- Imágenes en WebP, ancho máx. 1600 px, nombradas `01.webp`, `02.webp`...
- Masonry con CSS columns (sin librería).
- `next/image` con `sizes` y `placeholder="blur"` donde sea posible.
- Lightbox: shadcn `Dialog`, navegación con teclado y botones.
- Carga: las primeras 6 con `priority` / eager, el resto lazy.

## 10. Contacto

- `lib/whatsapp.js` → `buildWhatsAppUrl(locale)` devuelve `https://wa.me/<numero>?text=<mensaje en idioma>`.
- Número, usuario de Instagram y ciudades viven en `content/site.js`.
- Sin formularios ni backend.

## 11. SEO y rendimiento

- `sitemap.js` con los 4 locales y `alternates` (hreflang).
- Open Graph con imagen en `public/og.jpg`.
- JSON-LD `HealthAndBeautyBusiness` (nombre, ciudades, idiomas, enlace).
- Objetivo Lighthouse: 90+ en móvil (imágenes optimizadas, fuentes con `display: swap`).

## 12. Despliegue

- Repo en GitHub → proyecto en Vercel (deploy automático desde `main`).
- Demo en URL `*.vercel.app`; dominio propio después de la venta.
- Variables de entorno: ninguna obligatoria (WhatsApp e Instagram en `content/site.js`).

## 13. Contenido de `docs/`

| Archivo | Propósito |
|---|---|
| `README.md` | Índice y cómo correr el proyecto |
| `architecture.md` | Este documento |
| `design-system.md` | Tokens de color, fuentes, botones, espaciados |
| `i18n.md` | Cómo agregar/editar textos y un idioma nuevo, reglas RTL |
| `content-guide.md` | Cómo agregar fotos a la galería y optimizarlas |
| `deployment.md` | Pasos de Vercel y dominio |

## 14. Fuera de alcance (v1)

- CMS, panel de administración, base de datos.
- Reservas en línea, pagos, blog, tienda.
- Formulario de contacto con backend.
- Feed de Instagram en vivo.

## 15. Pendientes

- Fotos reales de trabajos (mín. 12–18) y retrato para About.
- Logo en SVG (el actual es imagen).
- Textos finales y revisión nativa de HE y RU.
- Confirmar servicios y si se muestran precios (por defecto: no).
