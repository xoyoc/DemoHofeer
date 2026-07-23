# RM Concreto Landing Page Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-page static landing site for RM Concreto (concrete construction, residential & commercial, Costa de Michoacán) using plain HTML/CSS/JS, styled with the logo's color palette, with the isotipo as favicon, full SEO, and WhatsApp as the sole contact CTA.

**Architecture:** Static files only (`index.html`, `styles.css`, `script.js`, `robots.txt`, `sitemap.xml`, `assets/`). No build step, no framework, no backend. Runs by opening `index.html` directly or serving the directory with any static host.

**Tech Stack:** HTML5, CSS3 (custom properties, Grid/Flexbox), vanilla JavaScript (ES5-compatible), ImageMagick (`convert`) for asset generation, Python 3 (Pillow) for verification scripts, Node.js for JS syntax checking.

## Global Constraints

- No build tools, package managers, or frameworks — plain HTML/CSS/JS only, per spec §3.
- No backend, no traditional contact form — WhatsApp link is the only contact mechanism, per spec §6 and the user's explicit choice.
- Language: `es` (Spanish, `es_MX` locale for Open Graph), per spec §7.
- Color palette is fixed to the 8 values in spec §4 — do not introduce new brand colors.
- Placeholder content (phone, email, address, social links, domain) must be clearly marked with `<!-- TODO -->` comments per spec §9 — these are intentional and must NOT be "fixed" by inventing fake-real data.
- Favicons are generated from `isotipo.png`; the header logo uses `logotipo_sf.png`.
- WhatsApp number used consistently everywhere: `528001234567` (placeholder, digits only, no `+`, for `wa.me` links).
- WhatsApp prefilled message used consistently everywhere: `Hola, me interesa información sobre sus servicios de construcción en concreto.` (URL-encoded as `Hola%2C%20me%20interesa%20informaci%C3%B3n%20sobre%20sus%20servicios%20de%20construcci%C3%B3n%20en%20concreto.`)
- Placeholder domain used consistently everywhere: `https://www.rmconcreto.mx`

---

### Task 1: Brand assets — favicons and Open Graph image

**Files:**
- Create: `assets/isotipo.png` (copy of source)
- Create: `assets/logotipo_sf.png` (copy of source)
- Create: `assets/favicon-16x16.png`
- Create: `assets/favicon-32x32.png`
- Create: `assets/favicon-180x180.png`
- Create: `assets/favicon.ico`
- Create: `assets/og-image.png`

**Interfaces:**
- Consumes: `isotipo.png` (421×330 RGBA) and `logotipo_sf.png` (546×457 RGBA) in the project root (already present).
- Produces: the `assets/` directory referenced by `index.html` in Task 2 — exact paths `assets/logotipo_sf.png`, `assets/isotipo.png`, `assets/favicon-16x16.png`, `assets/favicon-32x32.png`, `assets/favicon-180x180.png`, `assets/favicon.ico`, `assets/og-image.png`.

- [ ] **Step 1: Create the assets directory and copy source logos**

```bash
mkdir -p assets
cp isotipo.png assets/isotipo.png
cp logotipo_sf.png assets/logotipo_sf.png
```

- [ ] **Step 2: Generate favicon PNGs at required sizes from isotipo.png**

```bash
convert isotipo.png -resize 16x16 -background none -gravity center -extent 16x16 assets/favicon-16x16.png
convert isotipo.png -resize 32x32 -background none -gravity center -extent 32x32 assets/favicon-32x32.png
convert isotipo.png -resize 48x48 -background none -gravity center -extent 48x48 /tmp/favicon-48x48.png
convert isotipo.png -resize 180x180 -background white -gravity center -extent 180x180 assets/favicon-180x180.png
convert assets/favicon-16x16.png assets/favicon-32x32.png /tmp/favicon-48x48.png assets/favicon.ico
```

Note: `favicon-180x180.png` (apple-touch-icon) gets a white background instead of transparency — iOS renders transparent apple-touch-icons poorly.

- [ ] **Step 3: Generate the Open Graph preview image (1200×630, logo centered on light background)**

```bash
convert logotipo_sf.png -resize 700x700 /tmp/og-logo.png
convert -size 1200x630 xc:"#F8F9FA" /tmp/og-logo.png -gravity center -composite assets/og-image.png
```

- [ ] **Step 4: Verify all generated assets have the expected dimensions**

```bash
python3 - <<'EOF'
from PIL import Image
import os, sys

checks = {
    "assets/favicon-16x16.png": (16, 16),
    "assets/favicon-32x32.png": (32, 32),
    "assets/favicon-180x180.png": (180, 180),
    "assets/og-image.png": (1200, 630),
}

ok = True
for path, expected in checks.items():
    size = Image.open(path).size
    status = "PASS" if size == expected else "FAIL"
    if status == "FAIL":
        ok = False
    print(f"{status} {path}: {size} (expected {expected})")

if os.path.exists("assets/favicon.ico"):
    print("PASS assets/favicon.ico: exists")
else:
    print("FAIL assets/favicon.ico: missing")
    ok = False

sys.exit(0 if ok else 1)
EOF
```

Expected: every line prints `PASS`, script exits 0. If any line prints `FAIL`, re-run the `convert` command for that specific file before continuing.

- [ ] **Step 5: Commit**

```bash
git add assets/
git commit -m "Add brand assets: favicons and OG image generated from isotipo/logo"
```

---

### Task 2: index.html — semantic structure, SEO, and content

**Files:**
- Create: `index.html`

**Interfaces:**
- Consumes: `assets/logotipo_sf.png`, `assets/isotipo.png`, `assets/favicon-16x16.png`, `assets/favicon-32x32.png`, `assets/favicon-180x180.png`, `assets/favicon.ico`, `assets/og-image.png` from Task 1.
- Produces: `styles.css` link (consumed by Task 3), `script.js` link (consumed by Task 4), the following DOM contract Task 3/4 rely on:
  - `#header` (header element), `#nav` (nav element), `#navToggle` (button), `#year` (span in footer), `#main` (main element)
  - Section ids: `#inicio`, `#servicios`, `#nosotros`, `#contacto`
  - Classes referenced by JS: `.header--scrolled` (toggled on `#header`), `.nav--open` (toggled on `#nav`)

- [ ] **Step 1: Write index.html**

```html
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>RM Concreto | Construcción en Concreto Residencial y Comercial – Costa de Michoacán</title>
<meta name="description" content="RM Concreto: constructora especializada en concreto residencial y comercial en la Costa de Michoacán. Calidad, experiencia y garantía en cada obra.">
<meta name="keywords" content="concreto residencial, concreto comercial, construcción costa michoacana, constructora Michoacán, concreto Lázaro Cárdenas, obra civil">
<meta name="author" content="RM Concreto">
<link rel="canonical" href="https://www.rmconcreto.mx/">

<meta property="og:type" content="website">
<meta property="og:title" content="RM Concreto | Construcción en Concreto Residencial y Comercial">
<meta property="og:description" content="Constructora especializada en concreto residencial y comercial en la Costa de Michoacán. Calidad, experiencia y garantía en cada obra.">
<meta property="og:image" content="https://www.rmconcreto.mx/assets/og-image.png">
<meta property="og:url" content="https://www.rmconcreto.mx/">
<meta property="og:locale" content="es_MX">
<meta property="og:site_name" content="RM Concreto">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="RM Concreto | Construcción en Concreto Residencial y Comercial">
<meta name="twitter:description" content="Constructora especializada en concreto residencial y comercial en la Costa de Michoacán.">
<meta name="twitter:image" content="https://www.rmconcreto.mx/assets/og-image.png">

<link rel="icon" type="image/png" sizes="32x32" href="assets/favicon-32x32.png">
<link rel="icon" type="image/png" sizes="16x16" href="assets/favicon-16x16.png">
<link rel="apple-touch-icon" sizes="180x180" href="assets/favicon-180x180.png">
<link rel="shortcut icon" href="assets/favicon.ico">

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700;800&display=swap" rel="stylesheet">

<link rel="stylesheet" href="styles.css">

<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "GeneralContractor",
  "name": "RM Concreto",
  "image": "https://www.rmconcreto.mx/assets/logotipo_sf.png",
  "description": "Constructora especializada en concreto residencial y comercial en la Costa de Michoacán.",
  "telephone": "+52-800-123-4567",
  "email": "contacto@rmconcreto.mx",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Lázaro Cárdenas",
    "addressRegion": "Michoacán",
    "addressCountry": "MX"
  },
  "areaServed": "Costa de Michoacán",
  "url": "https://www.rmconcreto.mx/",
  "sameAs": [
    "https://facebook.com/rmconcreto",
    "https://instagram.com/rmconcreto"
  ]
}
</script>
</head>
<body>
<a href="#main" class="skip-link">Saltar al contenido principal</a>

<header class="header" id="header">
  <div class="container header__inner">
    <a href="#inicio" class="header__logo">
      <img src="assets/logotipo_sf.png" alt="RM Concreto - Construcción residencial y comercial" width="180" height="150">
    </a>
    <nav class="nav" id="nav">
      <ul class="nav__list">
        <li><a href="#inicio" class="nav__link">Inicio</a></li>
        <li><a href="#servicios" class="nav__link">Servicios</a></li>
        <li><a href="#nosotros" class="nav__link">Nosotros</a></li>
        <li><a href="#contacto" class="nav__link">Contacto</a></li>
        <li>
          <a href="https://wa.me/528001234567?text=Hola%2C%20me%20interesa%20informaci%C3%B3n%20sobre%20sus%20servicios%20de%20construcci%C3%B3n%20en%20concreto." class="btn btn--whatsapp" target="_blank" rel="noopener">
            <svg viewBox="0 0 24 24" width="18" height="18" fill="currentColor" aria-hidden="true"><path d="M12 2C6.48 2 2 6.48 2 12c0 1.85.5 3.58 1.38 5.07L2 22l5.06-1.35A9.94 9.94 0 0 0 12 22c5.52 0 10-4.48 10-10S17.52 2 12 2zm0 18c-1.6 0-3.1-.43-4.4-1.18l-.32-.19-3.02.8.81-2.94-.2-.32A7.94 7.94 0 0 1 4 12c0-4.41 3.59-8 8-8s8 3.59 8 8-3.59 8-8 8zm4.36-5.86c-.24-.12-1.42-.7-1.64-.78-.22-.08-.38-.12-.54.12-.16.24-.62.78-.76.94-.14.16-.28.18-.52.06-.24-.12-1.01-.37-1.92-1.18-.71-.63-1.19-1.41-1.33-1.65-.14-.24-.01-.37.11-.49.11-.11.24-.28.36-.42.12-.14.16-.24.24-.4.08-.16.04-.3-.02-.42-.06-.12-.54-1.3-.74-1.78-.19-.46-.39-.4-.54-.4-.14 0-.3-.02-.46-.02s-.42.06-.64.3c-.22.24-.84.82-.84 2s.86 2.32.98 2.48c.12.16 1.7 2.6 4.13 3.64.58.25 1.03.4 1.38.51.58.18 1.11.16 1.53.1.47-.07 1.42-.58 1.62-1.14.2-.56.2-1.04.14-1.14-.06-.1-.22-.16-.46-.28z"/></svg>
            WhatsApp
          </a>
        </li>
      </ul>
    </nav>
    <button class="nav__toggle" id="navToggle" aria-label="Abrir menú" aria-expanded="false" aria-controls="nav">
      <span></span><span></span><span></span>
    </button>
  </div>
</header>

<main id="main">
  <section class="hero" id="inicio">
    <div class="hero__bg" aria-hidden="true"></div>
    <div class="container hero__content">
      <h1 class="hero__title">Construcción en concreto residencial y comercial en la Costa de Michoacán</h1>
      <p class="hero__subtitle">Más de una década construyendo con calidad, precisión y materiales de primera. Casas, edificios, naves industriales y obra comercial con la garantía de un equipo especializado en concreto.</p>
      <div class="hero__actions">
        <a href="https://wa.me/528001234567?text=Hola%2C%20me%20interesa%20informaci%C3%B3n%20sobre%20sus%20servicios%20de%20construcci%C3%B3n%20en%20concreto." class="btn btn--whatsapp btn--lg" target="_blank" rel="noopener">
          <svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor" aria-hidden="true"><path d="M12 2C6.48 2 2 6.48 2 12c0 1.85.5 3.58 1.38 5.07L2 22l5.06-1.35A9.94 9.94 0 0 0 12 22c5.52 0 10-4.48 10-10S17.52 2 12 2zm0 18c-1.6 0-3.1-.43-4.4-1.18l-.32-.19-3.02.8.81-2.94-.2-.32A7.94 7.94 0 0 1 4 12c0-4.41 3.59-8 8-8s8 3.59 8 8-3.59 8-8 8zm4.36-5.86c-.24-.12-1.42-.7-1.64-.78-.22-.08-.38-.12-.54.12-.16.24-.62.78-.76.94-.14.16-.28.18-.52.06-.24-.12-1.01-.37-1.92-1.18-.71-.63-1.19-1.41-1.33-1.65-.14-.24-.01-.37.11-.49.11-.11.24-.28.36-.42.12-.14.16-.24.24-.4.08-.16.04-.3-.02-.42-.06-.12-.54-1.3-.74-1.78-.19-.46-.39-.4-.54-.4-.14 0-.3-.02-.46-.02s-.42.06-.64.3c-.22.24-.84.82-.84 2s.86 2.32.98 2.48c.12.16 1.7 2.6 4.13 3.64.58.25 1.03.4 1.38.51.58.18 1.11.16 1.53.1.47-.07 1.42-.58 1.62-1.14.2-.56.2-1.04.14-1.14-.06-.1-.22-.16-.46-.28z"/></svg>
          Cotiza por WhatsApp
        </a>
        <a href="#servicios" class="btn btn--outline btn--lg">Ver servicios</a>
      </div>
    </div>
  </section>

  <section class="services" id="servicios">
    <div class="container">
      <div class="section__header">
        <span class="section__eyebrow">Nuestros servicios</span>
        <h2 class="section__title">Concreto para cada tipo de proyecto</h2>
        <p class="section__subtitle">Desde una vivienda familiar hasta una obra comercial de gran escala, dominamos cada etapa del proceso constructivo.</p>
      </div>
      <div class="services__grid">
        <article class="service-card">
          <div class="service-card__image service-card__image--residential">
            <!-- TODO: reemplazar con foto real de proyecto residencial -->
            <span class="service-card__image-label">Foto próximamente</span>
          </div>
          <div class="service-card__icon">
            <svg viewBox="0 0 24 24" width="32" height="32" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M3 11l9-8 9 8"/><path d="M5 10v10h14V10"/><path d="M9 20v-6h6v6"/></svg>
          </div>
          <h3 class="service-card__title">Concreto Residencial</h3>
          <p class="service-card__text">Casas, fraccionamientos y remodelaciones con acabados en concreto de alta durabilidad, pensados para el clima costero.</p>
        </article>
        <article class="service-card">
          <div class="service-card__image service-card__image--commercial">
            <!-- TODO: reemplazar con foto real de proyecto comercial -->
            <span class="service-card__image-label">Foto próximamente</span>
          </div>
          <div class="service-card__icon">
            <svg viewBox="0 0 24 24" width="32" height="32" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="4" y="3" width="16" height="18" rx="1"/><path d="M9 21v-4h6v4"/><path d="M8 7h.01M12 7h.01M16 7h.01M8 11h.01M12 11h.01M16 11h.01M8 15h.01M16 15h.01"/></svg>
          </div>
          <h3 class="service-card__title">Concreto Comercial</h3>
          <p class="service-card__text">Locales, naves industriales y desarrollos comerciales con estructuras de concreto resistentes y de rápida ejecución.</p>
        </article>
        <article class="service-card">
          <div class="service-card__image service-card__image--finishes">
            <!-- TODO: reemplazar con foto real de acabados -->
            <span class="service-card__image-label">Foto próximamente</span>
          </div>
          <div class="service-card__icon">
            <svg viewBox="0 0 24 24" width="32" height="32" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="3" y="4" width="14" height="6" rx="1"/><path d="M8 10v4"/><path d="M8 18a2 2 0 1 0 4 0v-4H8z"/></svg>
          </div>
          <h3 class="service-card__title">Acabados</h3>
          <p class="service-card__text">Pulido, estampado y decorativo en concreto para pisos, fachadas y áreas exteriores con acabado profesional.</p>
        </article>
        <article class="service-card">
          <div class="service-card__image service-card__image--remodel">
            <!-- TODO: reemplazar con foto real de remodelación -->
            <span class="service-card__image-label">Foto próximamente</span>
          </div>
          <div class="service-card__icon">
            <svg viewBox="0 0 24 24" width="32" height="32" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M14.5 3.5l6 6L17 13l-6-6z"/><path d="M13 7l-8.5 8.5a2.12 2.12 0 0 0 3 3L16 10"/></svg>
          </div>
          <h3 class="service-card__title">Remodelación</h3>
          <p class="service-card__text">Ampliaciones, refuerzos estructurales y renovación de espacios existentes con la misma calidad de una obra nueva.</p>
        </article>
      </div>
    </div>
  </section>

  <section class="about" id="nosotros">
    <div class="container about__grid">
      <div class="about__content">
        <span class="section__eyebrow">Nosotros</span>
        <h2 class="section__title">Expertos en concreto en la costa michoacana</h2>
        <p class="about__text">En RM Concreto combinamos experiencia técnica y conocimiento local para entregar obras que resisten el clima costero y superan expectativas. Trabajamos de la mano con cada cliente, desde el diseño hasta la entrega final.</p>
        <ul class="about__features">
          <li class="about__feature">
            <span class="about__feature-icon"><svg viewBox="0 0 24 24" width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="8" r="5"/><path d="M8.5 12.5L7 22l5-3 5 3-1.5-9.5"/></svg></span>
            <div>
              <h3>Experiencia comprobada</h3>
              <p>Años de trayectoria en proyectos residenciales y comerciales en la región.</p>
            </div>
          </li>
          <li class="about__feature">
            <span class="about__feature-icon"><svg viewBox="0 0 24 24" width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/><path d="M9 12l2 2 4-4"/></svg></span>
            <div>
              <h3>Calidad de materiales</h3>
              <p>Concreto y agregados seleccionados para resistir la humedad y salinidad de la costa.</p>
            </div>
          </li>
          <li class="about__feature">
            <span class="about__feature-icon"><svg viewBox="0 0 24 24" width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 10c0 7-9 12-9 12s-9-5-9-12a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg></span>
            <div>
              <h3>Ubicación estratégica</h3>
              <p>Basados en la costa michoacana, con presencia y logística en toda la región.</p>
            </div>
          </li>
          <li class="about__feature">
            <span class="about__feature-icon"><svg viewBox="0 0 24 24" width="24" height="24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><circle cx="12" cy="12" r="10"/><path d="M8 12l3 3 5-6"/></svg></span>
            <div>
              <h3>Garantía de obra</h3>
              <p>Respaldamos cada proyecto con garantía por escrito y seguimiento post-entrega.</p>
            </div>
          </li>
        </ul>
      </div>
      <div class="about__visual" aria-hidden="true"></div>
    </div>
  </section>

  <section class="cta" id="contacto">
    <div class="cta__watermark" aria-hidden="true">
      <img src="assets/isotipo.png" alt="" width="300" height="235">
    </div>
    <div class="container cta__content">
      <h2 class="cta__title">¿Listo para construir tu próximo proyecto?</h2>
      <p class="cta__subtitle">Escríbenos por WhatsApp y recibe asesoría personalizada sin compromiso.</p>
      <a href="https://wa.me/528001234567?text=Hola%2C%20me%20interesa%20informaci%C3%B3n%20sobre%20sus%20servicios%20de%20construcci%C3%B3n%20en%20concreto." class="btn btn--whatsapp btn--lg" target="_blank" rel="noopener">
        <svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor" aria-hidden="true"><path d="M12 2C6.48 2 2 6.48 2 12c0 1.85.5 3.58 1.38 5.07L2 22l5.06-1.35A9.94 9.94 0 0 0 12 22c5.52 0 10-4.48 10-10S17.52 2 12 2zm0 18c-1.6 0-3.1-.43-4.4-1.18l-.32-.19-3.02.8.81-2.94-.2-.32A7.94 7.94 0 0 1 4 12c0-4.41 3.59-8 8-8s8 3.59 8 8-3.59 8-8 8zm4.36-5.86c-.24-.12-1.42-.7-1.64-.78-.22-.08-.38-.12-.54.12-.16.24-.62.78-.76.94-.14.16-.28.18-.52.06-.24-.12-1.01-.37-1.92-1.18-.71-.63-1.19-1.41-1.33-1.65-.14-.24-.01-.37.11-.49.11-.11.24-.28.36-.42.12-.14.16-.24.24-.4.08-.16.04-.3-.02-.42-.06-.12-.54-1.3-.74-1.78-.19-.46-.39-.4-.54-.4-.14 0-.3-.02-.46-.02s-.42.06-.64.3c-.22.24-.84.82-.84 2s.86 2.32.98 2.48c.12.16 1.7 2.6 4.13 3.64.58.25 1.03.4 1.38.51.58.18 1.11.16 1.53.1.47-.07 1.42-.58 1.62-1.14.2-.56.2-1.04.14-1.14-.06-.1-.22-.16-.46-.28z"/></svg>
        Escríbenos por WhatsApp
      </a>
      <div class="cta__info">
        <div class="cta__info-item">
          <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6 19.79 19.79 0 0 1-3.07-8.67A2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72c.127.96.361 1.903.7 2.81a2 2 0 0 1-.45 2.11L8.09 9.91a16 16 0 0 0 6 6l1.27-1.27a2 2 0 0 1 2.11-.45c.907.339 1.85.573 2.81.7A2 2 0 0 1 22 16.92z"/></svg>
          <div>
            <span class="cta__info-label">Teléfono</span>
            <!-- TODO: reemplazar con número real -->
            <a href="tel:+528001234567">+52 800 123 4567</a>
          </div>
        </div>
        <div class="cta__info-item">
          <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><rect x="2" y="4" width="20" height="16" rx="2"/><path d="M22 6l-10 7L2 6"/></svg>
          <div>
            <span class="cta__info-label">Correo</span>
            <!-- TODO: reemplazar con correo real -->
            <a href="mailto:contacto@rmconcreto.mx">contacto@rmconcreto.mx</a>
          </div>
        </div>
        <div class="cta__info-item">
          <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true"><path d="M21 10c0 7-9 12-9 12s-9-5-9-12a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>
          <div>
            <span class="cta__info-label">Ubicación</span>
            <!-- TODO: confirmar ciudad/dirección exacta -->
            <span>Costa de Michoacán, México</span>
          </div>
        </div>
      </div>
    </div>
  </section>
</main>

<footer class="footer">
  <div class="container footer__inner">
    <img src="assets/isotipo.png" alt="RM Concreto" class="footer__logo" width="48" height="38">
    <div class="footer__social">
      <!-- TODO: confirmar enlaces reales de redes sociales -->
      <a href="https://facebook.com/rmconcreto" target="_blank" rel="noopener" aria-label="Facebook de RM Concreto">
        <svg viewBox="0 0 24 24" width="20" height="20" fill="currentColor" aria-hidden="true"><path d="M13.5 9H15V6h-1.5C11.57 6 10 7.57 10 9.5V11H8v3h2v7h3v-7h2.1l.4-3H13v-1.1c0-.6.4-.9 1.5-.9z"/></svg>
      </a>
      <a href="https://instagram.com/rmconcreto" target="_blank" rel="noopener" aria-label="Instagram de RM Concreto">
        <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true"><rect x="3" y="3" width="18" height="18" rx="5"/><circle cx="12" cy="12" r="4"/><circle cx="17.5" cy="6.5" r="1"/></svg>
      </a>
    </div>
    <p class="footer__copy">&copy; <span id="year"></span> RM Concreto. Todos los derechos reservados.</p>
  </div>
</footer>

<script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify required SEO and structural elements are present**

```bash
grep -c "<h1" index.html
grep -q '<html lang="es">' index.html && echo "PASS lang" || echo "FAIL lang"
grep -q 'application/ld+json' index.html && echo "PASS jsonld" || echo "FAIL jsonld"
grep -q 'og:title' index.html && echo "PASS og" || echo "FAIL og"
grep -q 'name="description"' index.html && echo "PASS description" || echo "FAIL description"
grep -q 'rel="canonical"' index.html && echo "PASS canonical" || echo "FAIL canonical"
grep -q 'id="servicios"' index.html && echo "PASS servicios anchor" || echo "FAIL servicios anchor"
grep -q 'id="nosotros"' index.html && echo "PASS nosotros anchor" || echo "FAIL nosotros anchor"
grep -q 'id="contacto"' index.html && echo "PASS contacto anchor" || echo "FAIL contacto anchor"
```

Expected: the `<h1` count is exactly `1`, and every other line prints `PASS`.

- [ ] **Step 3: Verify structural tags are balanced and DOCTYPE is present**

```bash
python3 - <<'EOF'
import re
html = open("index.html", encoding="utf-8").read()
tags = ["header", "nav", "main", "section", "footer", "html", "body"]
ok = True
for tag in tags:
    opens = len(re.findall(rf"<{tag}[ >]", html))
    closes = len(re.findall(rf"</{tag}>", html))
    status = "PASS" if opens == closes and opens > 0 else "FAIL"
    if status == "FAIL":
        ok = False
    print(f"{status} <{tag}>: {opens} open / {closes} close")
if html.strip().lower().startswith("<!doctype html>"):
    print("PASS DOCTYPE present")
else:
    print("FAIL DOCTYPE missing")
    ok = False
import sys
sys.exit(0 if ok else 1)
EOF
```

Expected: all `PASS`, exit code 0.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add index.html with full SEO metadata and page content"
```

---

### Task 3: styles.css — design system and section styling

**Files:**
- Create: `styles.css`

**Interfaces:**
- Consumes: class names and ids defined in `index.html` (Task 2) — see the class list in Step 1 below.
- Produces: `styles.css` linked from `index.html`; defines `.header--scrolled` and `.nav--open` state classes that `script.js` (Task 4) toggles via `classList`.

- [ ] **Step 1: Write styles.css**

```css
:root {
  --color-red: #8B1E2F;
  --color-gray-light: #9CA3AF;
  --color-gray-dark: #4B5563;
  --color-gold: #F5A623;
  --color-orange: #E8590C;
  --color-turquoise: #0EA5C4;
  --color-blue-deep: #0B5D7A;
  --color-bg-light: #F8F9FA;
  --color-white: #FFFFFF;
  --color-text: #1F2937;
  --font-heading: 'Poppins', sans-serif;
  --font-body: 'Poppins', sans-serif;
  --container-width: 1180px;
  --radius: 12px;
  --shadow-md: 0 10px 30px rgba(11, 93, 122, 0.12);
  --transition: 0.25s ease;
}

*, *::before, *::after { box-sizing: border-box; }
html { scroll-behavior: smooth; }
body {
  margin: 0;
  font-family: var(--font-body);
  color: var(--color-text);
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}
img { max-width: 100%; display: block; }
a { text-decoration: none; color: inherit; }
ul { list-style: none; margin: 0; padding: 0; }
h1, h2, h3 { font-family: var(--font-heading); margin: 0 0 0.5em; line-height: 1.2; }
p { margin: 0 0 1em; }
:focus-visible { outline: 3px solid var(--color-gold); outline-offset: 2px; }

.skip-link {
  position: absolute;
  left: -9999px;
  top: 0;
  background: #fff;
  color: var(--color-blue-deep);
  padding: 12px 20px;
  z-index: 1000;
  border-radius: 0 0 8px 0;
  font-weight: 600;
}
.skip-link:focus { left: 0; top: 0; }

.container {
  width: 100%;
  max-width: var(--container-width);
  margin: 0 auto;
  padding: 0 24px;
}

.btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  border-radius: 999px;
  font-weight: 600;
  font-size: 0.95rem;
  transition: transform var(--transition), box-shadow var(--transition);
  cursor: pointer;
  border: none;
}
.btn--whatsapp {
  background: linear-gradient(135deg, var(--color-turquoise), var(--color-blue-deep));
  color: #fff;
  box-shadow: var(--shadow-md);
}
.btn--whatsapp:hover { transform: translateY(-2px); box-shadow: 0 14px 34px rgba(11, 93, 122, 0.22); }
.btn--outline { border: 2px solid var(--color-white); color: var(--color-white); }
.btn--outline:hover { background: rgba(255, 255, 255, 0.15); }
.btn--lg { padding: 16px 32px; font-size: 1.05rem; }

.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  background: var(--color-white);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
  padding: 14px 0;
  transition: box-shadow var(--transition), padding var(--transition);
}
.header--scrolled { box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); padding: 8px 0; }
.header__inner { display: flex; align-items: center; justify-content: space-between; gap: 24px; }
.header__logo img { height: 52px; width: auto; transition: height var(--transition); }
.header--scrolled .header__logo img { height: 42px; }

.nav__list { display: flex; align-items: center; gap: 32px; }
.nav__link { font-weight: 600; color: var(--color-blue-deep); transition: color var(--transition); }
.nav__link:hover { color: var(--color-turquoise); }

.nav__toggle {
  display: none;
  flex-direction: column;
  gap: 5px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 8px;
}
.nav__toggle span { width: 24px; height: 2px; background: var(--color-blue-deep); border-radius: 2px; }

@media (max-width: 900px) {
  .nav {
    position: fixed;
    top: 0;
    right: -100%;
    height: 100vh;
    width: min(320px, 80%);
    background: var(--color-white);
    padding: 100px 32px 32px;
    box-shadow: -10px 0 30px rgba(0, 0, 0, 0.1);
    transition: right var(--transition);
  }
  .nav--open { right: 0; }
  .nav__list { flex-direction: column; align-items: flex-start; gap: 24px; }
  .nav__toggle { display: flex; }
}

.hero {
  position: relative;
  padding: 160px 0 100px;
  background: linear-gradient(135deg, var(--color-gold) 0%, var(--color-orange) 45%, var(--color-turquoise) 100%);
  overflow: hidden;
}
.hero__bg {
  position: absolute;
  inset: 0;
  background-image: repeating-linear-gradient(45deg, rgba(255, 255, 255, 0.04) 0 2px, transparent 2px 40px);
  pointer-events: none;
}
.hero__content { position: relative; max-width: 760px; color: #fff; }
.hero__title {
  font-size: clamp(2rem, 4vw + 1rem, 3.2rem);
  font-weight: 800;
  color: #fff;
  text-shadow: 0 2px 20px rgba(0, 0, 0, 0.15);
}
.hero__subtitle { font-size: 1.1rem; color: rgba(255, 255, 255, 0.92); max-width: 620px; }
.hero__actions { display: flex; flex-wrap: wrap; gap: 16px; margin-top: 32px; }

.section__header { max-width: 680px; margin: 0 auto 56px; text-align: center; }
.section__eyebrow {
  display: inline-block;
  color: var(--color-turquoise);
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  font-size: 0.85rem;
  margin-bottom: 12px;
}
.section__title { font-size: clamp(1.6rem, 2.5vw + 1rem, 2.4rem); color: var(--color-blue-deep); }
.section__subtitle { color: var(--color-gray-dark); }

.services { padding: 100px 0; background: var(--color-bg-light); }
.services__grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 32px; }
.service-card {
  background: #fff;
  border-radius: var(--radius);
  overflow: hidden;
  box-shadow: var(--shadow-md);
  transition: transform var(--transition);
}
.service-card:hover { transform: translateY(-6px); }
.service-card__image {
  height: 160px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: rgba(255, 255, 255, 0.85);
  font-weight: 600;
  font-size: 0.9rem;
  text-align: center;
  padding: 0 16px;
}
.service-card__image--residential { background: linear-gradient(135deg, var(--color-red), var(--color-gray-dark)); }
.service-card__image--commercial { background: linear-gradient(135deg, var(--color-gold), var(--color-orange)); }
.service-card__image--finishes { background: linear-gradient(135deg, var(--color-turquoise), var(--color-blue-deep)); }
.service-card__image--remodel { background: linear-gradient(135deg, var(--color-gray-light), var(--color-gray-dark)); }
.service-card__icon { color: var(--color-turquoise); padding: 24px 24px 0; }
.service-card__title { padding: 8px 24px 0; color: var(--color-blue-deep); font-size: 1.2rem; }
.service-card__text { padding: 8px 24px 28px; color: var(--color-gray-dark); font-size: 0.95rem; }

.about { padding: 100px 0; }
.about__grid { display: grid; grid-template-columns: 1.2fr 1fr; gap: 56px; align-items: center; }
.about__text { color: var(--color-gray-dark); font-size: 1.05rem; }
.about__features { display: grid; gap: 24px; margin-top: 32px; }
.about__feature { display: flex; gap: 16px; align-items: flex-start; }
.about__feature-icon {
  flex-shrink: 0;
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: rgba(14, 165, 196, 0.1);
  color: var(--color-turquoise);
  display: flex;
  align-items: center;
  justify-content: center;
}
.about__feature h3 { font-size: 1.05rem; color: var(--color-blue-deep); margin-bottom: 4px; }
.about__feature p { color: var(--color-gray-dark); font-size: 0.92rem; margin: 0; }
.about__visual {
  min-height: 320px;
  border-radius: var(--radius);
  background: linear-gradient(135deg, var(--color-turquoise), var(--color-blue-deep), var(--color-gold));
}
@media (max-width: 900px) {
  .about__grid { grid-template-columns: 1fr; }
  .about__visual { display: none; }
}

.cta {
  position: relative;
  padding: 100px 0;
  background: linear-gradient(135deg, var(--color-blue-deep), var(--color-turquoise));
  color: #fff;
  overflow: hidden;
}
.cta__watermark { position: absolute; right: -40px; bottom: -40px; opacity: 0.12; width: 320px; pointer-events: none; }
.cta__content { position: relative; text-align: center; max-width: 680px; margin: 0 auto; }
.cta__title { color: #fff; font-size: clamp(1.6rem, 2.5vw + 1rem, 2.4rem); }
.cta__subtitle { color: rgba(255, 255, 255, 0.9); margin-bottom: 32px; }
.cta__info { display: flex; flex-wrap: wrap; justify-content: center; gap: 32px; margin-top: 48px; }
.cta__info-item { display: flex; align-items: center; gap: 12px; text-align: left; }
.cta__info-label {
  display: block;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: rgba(255, 255, 255, 0.7);
}
.cta__info-item a, .cta__info-item span { color: #fff; font-weight: 600; }

.footer { background: var(--color-gray-dark); color: rgba(255, 255, 255, 0.85); padding: 40px 0; }
.footer__inner { display: flex; flex-direction: column; align-items: center; gap: 16px; text-align: center; }
.footer__logo { width: 48px; opacity: 0.9; }
.footer__social { display: flex; gap: 16px; }
.footer__social a { color: #fff; opacity: 0.85; transition: opacity var(--transition); }
.footer__social a:hover { opacity: 1; }
.footer__copy { font-size: 0.85rem; margin: 0; }

@media (max-width: 600px) {
  .hero { padding: 140px 0 72px; }
  .services, .about, .cta { padding: 64px 0; }
}
```

- [ ] **Step 2: Verify required custom properties and key selectors exist**

```bash
grep -q -- "--color-turquoise" styles.css && echo "PASS turquoise var" || echo "FAIL turquoise var"
grep -q -- "--color-red" styles.css && echo "PASS red var" || echo "FAIL red var"
grep -q ".header--scrolled" styles.css && echo "PASS header--scrolled" || echo "FAIL header--scrolled"
grep -q ".nav--open" styles.css && echo "PASS nav--open" || echo "FAIL nav--open"
grep -q "@media" styles.css && echo "PASS media queries" || echo "FAIL media queries"
```

Expected: all lines print `PASS`.

- [ ] **Step 3: Serve the site locally and confirm styles.css loads with a 200 status**

```bash
python3 -m http.server 8123 --directory . &
SERVER_PID=$!
sleep 1
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8123/styles.css
kill $SERVER_PID
```

Expected: `200`.

- [ ] **Step 4: Commit**

```bash
git add styles.css
git commit -m "Add styles.css with RM Concreto design system and responsive layout"
```

---

### Task 4: script.js — interactivity

**Files:**
- Create: `script.js`

**Interfaces:**
- Consumes: `#header`, `#nav`, `#navToggle`, `#year` ids from `index.html` (Task 2); `.header--scrolled` and `.nav--open` classes from `styles.css` (Task 3).
- Produces: no exports — this is the final file in the dependency chain.

- [ ] **Step 1: Write script.js**

```javascript
document.addEventListener('DOMContentLoaded', function () {
  var header = document.getElementById('header');
  var nav = document.getElementById('nav');
  var navToggle = document.getElementById('navToggle');
  var yearEl = document.getElementById('year');

  if (yearEl) {
    yearEl.textContent = new Date().getFullYear();
  }

  function onScroll() {
    if (window.scrollY > 20) {
      header.classList.add('header--scrolled');
    } else {
      header.classList.remove('header--scrolled');
    }
  }
  window.addEventListener('scroll', onScroll);
  onScroll();

  navToggle.addEventListener('click', function () {
    var isOpen = nav.classList.toggle('nav--open');
    navToggle.setAttribute('aria-expanded', String(isOpen));
  });

  var navLinks = nav.querySelectorAll('a');
  for (var i = 0; i < navLinks.length; i++) {
    navLinks[i].addEventListener('click', function () {
      nav.classList.remove('nav--open');
      navToggle.setAttribute('aria-expanded', 'false');
    });
  }
});
```

- [ ] **Step 2: Verify JS syntax is valid**

```bash
node --check script.js
```

Expected: no output, exit code 0.

- [ ] **Step 3: Verify key behaviors are implemented**

```bash
grep -q "header--scrolled" script.js && echo "PASS scroll behavior" || echo "FAIL scroll behavior"
grep -q "nav--open" script.js && echo "PASS mobile menu" || echo "FAIL mobile menu"
grep -q "getFullYear" script.js && echo "PASS dynamic year" || echo "FAIL dynamic year"
```

Expected: all lines print `PASS`.

- [ ] **Step 4: Commit**

```bash
git add script.js
git commit -m "Add script.js for mobile menu, scroll state, and dynamic year"
```

---

### Task 5: robots.txt and sitemap.xml

**Files:**
- Create: `robots.txt`
- Create: `sitemap.xml`

**Interfaces:**
- Consumes: placeholder domain `https://www.rmconcreto.mx` (Global Constraints).
- Produces: no code interfaces — these are standalone SEO discovery files.

- [ ] **Step 1: Write robots.txt**

```
User-agent: *
Allow: /

Sitemap: https://www.rmconcreto.mx/sitemap.xml
```

- [ ] **Step 2: Write sitemap.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://www.rmconcreto.mx/</loc>
    <lastmod>2026-07-23</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

- [ ] **Step 3: Validate sitemap.xml is well-formed XML**

```bash
python3 -c "import xml.etree.ElementTree as ET; ET.parse('sitemap.xml'); print('PASS: sitemap.xml is valid XML')"
```

Expected: `PASS: sitemap.xml is valid XML`.

- [ ] **Step 4: Verify robots.txt references the sitemap**

```bash
grep -q "Sitemap: https://www.rmconcreto.mx/sitemap.xml" robots.txt && echo "PASS" || echo "FAIL"
```

Expected: `PASS`.

- [ ] **Step 5: Commit**

```bash
git add robots.txt sitemap.xml
git commit -m "Add robots.txt and sitemap.xml for search engine discovery"
```

---

### Task 6: Integration verification and final review

**Files:**
- None created — this task verifies the combined output of Tasks 1–5.

**Interfaces:**
- Consumes: all files produced by Tasks 1–5.
- Produces: none.

- [ ] **Step 1: Serve the full site locally and verify key routes return 200**

```bash
python3 -m http.server 8123 --directory . &
SERVER_PID=$!
sleep 1
curl -s http://localhost:8123/ | grep -o "<title>.*</title>"
curl -s -o /dev/null -w "index.html: %{http_code}\n" http://localhost:8123/
curl -s -o /dev/null -w "styles.css: %{http_code}\n" http://localhost:8123/styles.css
curl -s -o /dev/null -w "script.js: %{http_code}\n" http://localhost:8123/script.js
curl -s -o /dev/null -w "favicon.ico: %{http_code}\n" http://localhost:8123/assets/favicon.ico
curl -s -o /dev/null -w "og-image.png: %{http_code}\n" http://localhost:8123/assets/og-image.png
curl -s -o /dev/null -w "robots.txt: %{http_code}\n" http://localhost:8123/robots.txt
curl -s -o /dev/null -w "sitemap.xml: %{http_code}\n" http://localhost:8123/sitemap.xml
kill $SERVER_PID
```

Expected: the title line contains `RM Concreto | Construcción en Concreto Residencial y Comercial – Costa de Michoacán`, and every route prints `200`.

- [ ] **Step 2: Visual and interaction check in a real browser (desktop and mobile widths)**

Using the browser automation tool available in the session (e.g. Claude in Chrome), with the local server from Step 1 still running (or restarted):
1. Navigate to `http://localhost:8123/`.
2. At desktop width (≥1280px): confirm the header, hero, services grid (4 cards), about section, CTA section, and footer all render with the RM Concreto palette (red, gray, gold/orange, turquoise) and no layout overlap.
3. Scroll down and confirm the header gains a shadow/shrinks (the `header--scrolled` state).
4. Resize to a mobile width (~375px): confirm the hamburger button appears, the desktop nav list is hidden, clicking the hamburger opens the nav panel, and clicking a nav link closes it again.
5. Confirm all three WhatsApp buttons (header, hero, CTA section) point to a `wa.me` link.
6. Read console messages and confirm there are no JavaScript errors.

This step is manual/exploratory and should be run directly in the primary session (not delegated to a subagent without browser access).

- [ ] **Step 3: Confirm git status is clean**

```bash
git status
```

Expected: `nothing to commit, working tree clean` (after Step 2's manual check, no code changes are expected; if issues were found and fixed, commit them first with a descriptive message before this final check).
