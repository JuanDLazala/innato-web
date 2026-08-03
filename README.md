# INNATO — Deploy

Sitio estático del medio digital ecuatoriano INNATO.

## Estructura

```
deploy/
├── index.html              # Página principal (HTML + Tailwind CDN + React Babel)
├── netlify.toml            # Configuración de Netlify
├── .gitignore
└── assets_stable/
    ├── logo.jpg
    ├── nico-main.jpg
    └── social/
        └── feed-1.jpg
```

## Cómo desplegar (GitHub + Netlify)

### Paso 1 — Crear el repositorio en GitHub
1. Entra a https://github.com/new
2. Nombre sugerido: `innato-web`
3. Visibilidad: Public o Private (lo que prefieras)
4. **No** marques "Initialize with README" (este folder ya tiene los archivos)
5. Clic en **Create repository**

### Paso 2 — Subir esta carpeta a GitHub
Abre PowerShell o Terminal dentro de esta carpeta `deploy` y ejecuta:

```bash
git init
git add .
git commit -m "Initial commit: INNATO landing"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/innato-web.git
git push -u origin main
```

Reemplaza `TU_USUARIO` por tu usuario real de GitHub.

### Paso 3 — Conectar Netlify
1. Entra a https://app.netlify.com/
2. Clic en **Add new site → Import an existing project**
3. Elige **Deploy with GitHub** y autoriza el acceso
4. Selecciona el repo `innato-web`
5. En la configuración de build:
   - **Branch**: `main`
   - **Build command**: (déjalo vacío)
   - **Publish directory**: `.`
6. Clic en **Deploy site**

En 30–60 segundos tendrás una URL tipo `https://random-name-12345.netlify.app`.

### Paso 4 — Personalizar el subdominio
1. En el dashboard del site → **Site configuration → Change site name**
2. Pon algo como `innato` o `innato-media` → quedará `https://innato.netlify.app`

### Despliegues posteriores
Cada `git push` a `main` redeploya automáticamente. Para actualizar la página solo:

```bash
git add .
git commit -m "Descripción del cambio"
git push
```

## Cómo actualizar el feed (sección "Lo último en INNATO")

Todo el contenido dinámico vive en **`feed.json`** (raíz del proyecto). Cambias el JSON y haces push, Netlify redeploya solo en ~30 seg.

### Estructura

```json
{
  "social": {
    "tiktok":    { "url": "...", "title": "...", "thumbnail": "assets_stable/social/...", "date": "Hoy" },
    "instagram": { "url": "...", "title": "...", "thumbnail": "...", "date": "Ayer" },
    "facebook":  { "url": "...", "title": "...", "thumbnail": "...", "date": "hace 2 días" }
  },
  "blog": [
    {
      "slug": "primer-post",
      "title": "Título del post",
      "excerpt": "Resumen breve...",
      "date": "2026-05-08",
      "thumbnail": "assets_stable/blog/primer-post.jpg",
      "content": "Contenido completo (Markdown o HTML)..."
    }
  ]
}
```

### Workflow para una nueva publicación de TikTok / Instagram / Facebook

1. Tomar screenshot del post (1280×720 aprox).
2. Guardarlo en `assets_stable/social/` (ej. `tiktok-2026-05-12.jpg`).
3. Editar `feed.json`: actualizar `url`, `title`, `thumbnail` y `date` de la red correspondiente.
4. `git add . && git commit -m "feed: nuevo post de TikTok" && git push`.

### Workflow para un nuevo post de blog

Cada entrada tiene **su propia página y su propia URL** (`innatoec.com/blog/<slug>/`), con metadatos propios para compartir. Eso obliga a un paso extra: generar los archivos.

**1. Agregar la entrada al array `blog` de `feed.json`:**

```json
{
  "slug": "url-de-la-nota",
  "title": "Titular de la nota",
  "excerpt": "Bajada de una o dos frases.",
  "date": "2026-08-03",
  "category": "Ecuador en contexto",
  "author": "Redacción INNATOec",
  "readingTime": "4 min",
  "nivel": "VERIFICADO",
  "fuente": "El Universo · Primicias",
  "thumbnail": "assets_stable/innato-hero.jpg",
  "content": "<p>Cuerpo en HTML. Usar &lt;h3&gt; para subtítulos.</p>"
}
```

`nivel` es obligatorio: `VERIFICADO`, `DECLARACIÓN` o `EN DESARROLLO`. Ver la sección 6 del brand book.

**2. Generar las páginas y los artes de compartir:**

```
cd ..
python scripts/generar_og_notas.py
python scripts/generar_blog.py
cd deploy
```

Esto crea `blog/<slug>/index.html`, regenera `blog/index.html` y produce `assets_stable/og/<slug>.jpg` con el titular sobre la plancha lima.

**3. Publicar:**

```
git add . && git commit -m "blog: nueva entrada X" && git push
```

> **Por qué páginas estáticas y no rutas de JavaScript**: los rastreadores de WhatsApp, Facebook y X no ejecutan JS. Sin un HTML real por nota, todas compartirían la misma tarjeta y la pauta publicitaria no podría segmentar por nota.

## Notas técnicas
- El sitio usa Tailwind y React desde CDN (sin build step). Para producción seria conviene migrar a un build estático, pero para el MVP esto funciona perfecto.
- `netlify.toml` incluye headers de seguridad, cache largo (1 año) para `/assets_stable/*` y cache corto (60 seg) para `/feed.json`.
