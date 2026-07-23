# Landing Page — RM Concreto

**Fecha:** 2026-07-23
**Estado:** Aprobado

## 1. Resumen

Landing page de una sola página para RM Concreto, constructora especializada en
concreto residencial y comercial ubicada en la costa michoacana. Sitio estático
(HTML + CSS + JS puro, sin build ni backend), optimizado para SEO, con la
paleta de colores extraída del logotipo y el isotipo usado como favicon.

## 2. Objetivo y alcance

- Presentar la marca RM Concreto y sus servicios de construcción.
- Generar contacto directo vía WhatsApp (sin formulario tradicional).
- Página corta, un solo scroll — sin backend, sin dependencias de build.
- Fuera de alcance: blog, CMS, múltiples páginas, formulario con servicio
  externo, fotos reales de proyectos (se dejan placeholders).

## 3. Arquitectura

Archivos estáticos en la raíz del proyecto:

```
/index.html
/styles.css
/script.js
/assets/
  logotipo_sf.png       (logo horizontal, ya existente)
  isotipo.png           (ya existente, fuente para favicons)
  favicon-16x16.png
  favicon-32x32.png
  favicon-180x180.png   (apple-touch-icon)
  favicon.ico
  og-image.png          (imagen para Open Graph, basada en logotipo_sf.png)
/robots.txt
/sitemap.xml
```

Sin frameworks, sin gestor de paquetes. Se puede abrir `index.html`
directamente o desplegar en cualquier hosting estático (Netlify, GitHub Pages,
hosting compartido tradicional).

Los favicons se generan a partir de `isotipo.png` en los tamaños estándar
listados arriba.

## 4. Paleta de colores

Extraída del logotipo (`logotipo_sf.png`):

| Uso | Color | Hex |
|---|---|---|
| Rojo concreto (vino) | acentos, CTA secundario, detalles de la "R" | `#8B1E2F` |
| Gris concreto claro | fondos neutros, texturas | `#9CA3AF` |
| Gris concreto oscuro | texto secundario, fondos oscuros | `#4B5563` |
| Dorado (sol) | acentos cálidos, iconos destacados | `#F5A623` |
| Naranja (sol, degradado) | degradados cálidos | `#E8590C` |
| Turquesa (ola) | CTA principal, enlaces, detalles | `#0EA5C4` |
| Azul profundo (ola, degradado) | degradados fríos, texto "CONCRETO" | `#0B5D7A` |
| Blanco / gris muy claro | fondos de sección alternados | `#F8F9FA` |

Variables CSS (`:root`) centralizan estos valores para reutilización
consistente en todo el sitio.

## 5. Contenido y secciones

Todas las secciones en `index.html`, en este orden:

1. **Header fijo**
   - Logo horizontal (`logotipo_sf.png`) a la izquierda.
   - Menú: Inicio, Servicios, Nosotros, Contacto (scroll suave a anclas).
   - Botón CTA "WhatsApp" destacado en turquesa, visible en escritorio y
     colapsado a icono en móvil.

2. **Hero**
   - `<h1>`: "Construcción en concreto residencial y comercial en la Costa de
     Michoacán".
   - Subtítulo breve reforzando experiencia y calidad.
   - CTA principal: botón grande a WhatsApp con mensaje predefinido
     (`https://wa.me/<PLACEHOLDER_TELEFONO>?text=...`).
   - Fondo con degradado de la paleta (dorado → turquesa) y textura sutil de
     concreto (CSS, sin imagen pesada).

3. **Servicios**
   - Grid de 3–4 tarjetas: Concreto residencial, Concreto comercial,
     Acabados, Remodelación.
   - Cada tarjeta: icono (SVG inline simple, sin librería externa), título,
     descripción breve.
   - Placeholder de imagen por tarjeta (`<!-- TODO: reemplazar con foto de
     proyecto real -->`), con estilo de bloque de color de la paleta mientras
     tanto.

4. **Nosotros / Por qué elegirnos**
   - Texto breve sobre la empresa.
   - 3–4 puntos clave con iconos: experiencia, calidad de materiales,
     ubicación estratégica en la costa michoacana, garantía de obra.

5. **CTA final + Contacto**
   - Botón grande a WhatsApp (mismo mecanismo que el hero).
   - Datos de contacto en placeholders claramente marcados:
     - Teléfono: `+52 XXX XXX XXXX`
     - Email: `contacto@rmconcreto.com.mx`
     - Ubicación: `Costa de Michoacán, México`
   - Isotipo (`isotipo.png`) como elemento decorativo grande, con opacidad
     baja o como marca de agua.

6. **Footer**
   - Isotipo pequeño.
   - Enlaces a redes sociales (placeholders: Facebook, Instagram).
   - Copyright con año dinámico (JS) y nombre de la empresa.

## 6. Interactividad (script.js)

- Scroll suave a anclas del menú.
- Menú móvil tipo hamburguesa (toggle simple de clase CSS).
- Header con cambio de estilo al hacer scroll (sombra/fondo sólido).
- Año dinámico en el footer.
- Botones de WhatsApp: enlaces `<a href="https://wa.me/...">`, sin
  JavaScript adicional necesario para el envío.

## 7. SEO

- `<title>` único y descriptivo con palabras clave (concreto residencial,
  comercial, costa michoacana, construcción).
- `<meta name="description">` optimizada (150–160 caracteres).
- `<meta name="keywords">` con términos relevantes.
- Open Graph (`og:title`, `og:description`, `og:image`, `og:type`,
  `og:locale` = `es_MX`) y Twitter Card (`summary_large_image`).
- JSON-LD `LocalBusiness` (subtipo `GeneralContractor`) con nombre,
  dirección (placeholder), teléfono (placeholder), área de servicio
  (costa michoacana), imagen de logo.
- HTML semántico: un solo `<h1>`, `<header>`, `<main>`, `<section>` con
  `id`s descriptivos, `<footer>`.
- Atributos `alt` descriptivos en todas las imágenes (logo, isotipo,
  placeholders de servicios).
- `robots.txt` permitiendo indexación completa + referencia a
  `sitemap.xml`.
- `sitemap.xml` con la URL única de la landing (placeholder de dominio,
  editable).
- Favicon completo (16x16, 32x32, apple-touch-icon, `.ico`) generado desde
  `isotipo.png`.
- Idioma declarado: `<html lang="es">`.

## 8. Responsive / accesibilidad

- Mobile-first, breakpoints simples (móvil, tablet ~768px, escritorio
  ~1024px).
- Contraste de color verificado entre texto y fondos de la paleta.
- Menú móvil accesible por teclado (foco visible, `aria-expanded` en el
  botón hamburguesa).
- Imágenes con `alt`; botones con texto o `aria-label` claro.

## 9. Placeholders a reemplazar por el usuario

Marcados con comentarios `<!-- TODO -->` en el HTML:

- Número de WhatsApp (usado en 2 botones).
- Email de contacto.
- Dirección/ciudad específica en la costa michoacana.
- Enlaces a redes sociales.
- Dominio real en `sitemap.xml`, `og:url`, y JSON-LD.
- Fotos reales de proyectos (reemplazan los bloques de color en la sección
  de Servicios).

## 10. Fuera de alcance (explícito)

- Formulario de contacto tradicional / integración con Formspree.
- Galería de proyectos ampliada o multi-página.
- Blog o CMS.
- Analítica (Google Analytics, Meta Pixel) — no solicitado; se puede añadir
  después si se pide.
