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

1. Agregar un objeto **al inicio** del array `blog` en `feed.json` (el primero del array es el destacado en la home).
2. Si tiene imagen, guardarla en `assets_stable/blog/` y referenciarla en `thumbnail`.
3. `git add . && git commit -m "blog: nuevo post X" && git push`.

> El detalle del post (página individual al hacer click en una tarjeta) se implementa en una segunda iteración. Por ahora la tarjeta lleva a `#blog/<slug>`.

## Notas técnicas
- El sitio usa Tailwind y React desde CDN (sin build step). Para producción seria conviene migrar a un build estático, pero para el MVP esto funciona perfecto.
- `netlify.toml` incluye headers de seguridad, cache largo (1 año) para `/assets_stable/*` y cache corto (60 seg) para `/feed.json`.
