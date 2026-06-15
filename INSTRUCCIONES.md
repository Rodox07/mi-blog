# Cómo poner tu blog online (paso a paso)

Este blog usa **Eleventy** (genera las páginas), **Decap CMS** (panel de login para crear entradas) y se publica gratis con **GitHub Pages**. El login pasa por **Netlify** (gratis, solo se usa para la autenticación, el sitio NO vive ahí).

No necesitás saber programar para seguir estos pasos. Vas a tardar unos 20-30 minutos la primera vez. Después de esto, crear una reseña o un mensaje te toma 1 minuto.

---

## Paso 1 — Crear el repositorio en GitHub

1. Entrá a [github.com](https://github.com) y creá una cuenta si no tenés.
2. Click en **New repository** (botón verde, arriba a la derecha).
3. Nombre del repo: por ejemplo `mi-blog` (sin espacios).
4. Marcalo como **Public**.
5. NO marques "Add a README" (vamos a subir los archivos ya armados).
6. Click en **Create repository**.

---

## Paso 2 — Subir estos archivos al repositorio

La forma más fácil sin usar la terminal:

1. En la página de tu repo vacío, vas a ver un link que dice **uploading an existing file**. Click ahí.
2. Arrastrá **todos** los archivos y carpetas de este proyecto (`src/`, `css/`, `js/`, `images/`, `admin/`, `.eleventy.js`, `package.json`, y este mismo archivo).
3. Abajo, escribí un mensaje como "primer commit" y click en **Commit changes**.

> Tip: si tu repo se llama distinto a `mi-blog`, no importa, solo recordá el nombre para el paso 6.

---

## Paso 3 — Conectar el repo a Netlify (solo para el login)

1. Entrá a [netlify.com](https://netlify.com) y creá una cuenta gratis usando **"Sign up with GitHub"**.
2. Una vez dentro, click en **Add new site → Import an existing project**.
3. Elegí **GitHub** y autorizá el acceso.
4. Seleccioná tu repositorio `mi-blog`.
5. En la configuración de build, poné:
   - **Build command**: `npx @11ty/eleventy`
   - **Publish directory**: `_site`
6. Click en **Deploy site**. Netlify va a generar una URL tipo `algo-random.netlify.app` — no importa, no vamos a usar esa URL para el blog final, solo para el login.

---

## Paso 4 — Activar el login (Netlify Identity + Git Gateway)

1. Dentro del sitio en Netlify, vas a la pestaña **Identity** (en el menú de arriba).
2. Click en **Enable Identity**.
3. Bajá hasta **Registration** y elegí **Invite only** (así solo vos y tu novia pueden entrar, nadie más).
4. Bajá hasta **Services → Git Gateway** y click en **Enable Git Gateway**. Esto le da permiso a Decap CMS para hacer commits en tu nombre.
5. Volvé arriba a la pestaña **Identity** y click en **Invite users**. Escribí tu email (y el de tu novia si querés que ella también pueda postear). Les va a llegar un mail para crear su contraseña.

---

## Paso 5 — Activar GitHub Pages

1. En tu repo de GitHub, ir a **Settings → Pages**.
2. En "Build and deployment", source: **GitHub Actions** (recomendado) — GitHub te va a sugerir un workflow para Eleventy, o podés usar el que dejamos en `.github/workflows/deploy.yml` (ver Paso 7).
3. Esperá un par de minutos y vas a ver la URL de tu sitio, algo como `https://TU-USUARIO.github.io/mi-blog/`.

---

## Paso 6 — Ajustar la URL en `admin/config.yml`

Abrí el archivo `admin/config.yml` (podés editarlo directo en GitHub, click en el lápiz ✏️) y reemplazá estas dos líneas con tu URL real de GitHub Pages:

```yaml
site_url: https://TU-USUARIO.github.io/TU-REPOSITORIO
display_url: https://TU-USUARIO.github.io/TU-REPOSITORIO
```

Guardá los cambios (Commit changes).

---

## Paso 7 — Workflow de GitHub Actions (build automático)

Creá el archivo `.github/workflows/deploy.yml` con este contenido (podés crearlo directo desde GitHub: "Add file → Create new file" y pegar la ruta completa incluyendo las carpetas):

```yaml
name: Deploy blog
on:
  push:
    branches: ["main"]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: npx eleventy
      - uses: actions/upload-pages-artifact@v3
        with:
          path: _site
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

Cada vez que cambies algo (manualmente o desde el panel de admin), esto reconstruye y publica el sitio automáticamente.

---

## Paso 8 — ¡Usar el panel!

1. Entrá a `https://TU-USUARIO.github.io/TU-REPOSITORIO/admin/`
2. Click en **Login with Netlify Identity**.
3. Iniciá sesión con el email/contraseña que configuraste en el Paso 4.
4. Vas a ver dos secciones:
   - **Reseñas de discos**: click en "New Reseña", completá título, artista, año, estrellitas, tapa y tu texto.
   - **Mensajes del día**: click en "New Mensaje", escribí el mensaje y subí o pegá el link del gif.
5. Click en **Publish** (o "Save" según versión). Eso hace un commit al repo, dispara el build de GitHub Actions, y en 1-2 minutos tu blog se actualiza solo.

---

## Personalización rápida

- **Colores**: editá las variables al inicio de `css/estilos.css` (sección `:root`).
- **Textos del header**: editá `src/_includes/base.njk`.
- **Mensaje del marquee**: también en `base.njk`, dentro de `<marquee>`.

---

## ¿Problemas comunes?

- **El login no aparece / da error**: revisá que Identity y Git Gateway estén activados en Netlify (Paso 4), y que hayas aceptado la invitación por mail.
- **El sitio no se actualiza después de publicar**: revisá la pestaña "Actions" en GitHub para ver si el workflow corrió bien.
- **Las imágenes no se ven**: asegurate de subirlas desde el propio panel (botón de imagen), así quedan guardadas en `images/subidas/` dentro del repo.
