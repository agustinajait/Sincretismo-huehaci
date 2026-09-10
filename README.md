# Sincretismo — Huehaci

Herramienta para que las organizaciones sociales acompañadas por Huehaci carguen,
de forma sincronizada en vivo, su nómina de colaboradores, datos generales de
equipo, servicios/infraestructura y diagnóstico institucional (con adjuntos).

## Estructura

- `index.html` — landing de presentación del proyecto (root del sitio). Tiene
  dos accesos:
  - **Admin** (🔐 en la barra superior): pide una contraseña y, si es correcta,
    redirige a `diagnostico.html?admin=TOKEN` con el token de administrador
    hardcodeado en el propio `index.html`.
  - **Organización** ("Hacer tu diagnóstico"): pide un código de acceso propio
    de cada organización, lo valida contra Supabase y redirige a
    `diagnostico.html?org=N&token=X`.
- `diagnostico.html` — la herramienta en sí: tablas, formularios y sincronización
  en vivo vía Supabase (Realtime + polling de respaldo). Lee los parámetros de
  la URL (`?admin=` o `?org=&token=`) para decidir qué organización(es) mostrar
  y en qué modo (administrador o solo esa organización).

Ambos archivos son estáticos (HTML + CSS + JS inline, sin build step) y cargan
las librerías externas (Supabase JS, SheetJS) desde CDN.

## Backend (Supabase)

- **Project ref**: `qwnrdclaeaskoyskbqff`
- **Tabla**: `sincretismo_nomina`
  - `org_index` (int), `access_token` (text)
  - `org_name`, `rows`, `general`, `servicios`, `diagnostico` — todas `jsonb`
  - `updated_at`
- **Storage bucket**: `sincretismo-adjuntos` (público) — archivos adjuntos del
  diagnóstico institucional.

La clave usada en el cliente es la **publishable key** de Supabase (segura para
exponer en el frontend); el control de acceso real de cada organización lo da
el `access_token` propio guardado en la tabla, no la clave de Supabase. El
token de administrador y los tokens por organización están hardcodeados en el
código de `index.html` / `diagnostico.html`.

## Despliegue (Netlify vía Git)

Este repo está pensado para conectarse directo a Netlify por Git, sin subir
carpetas a mano:

1. Conectar el repo de GitHub en Netlify (**Add new site → Import an existing
   project**).
2. Build settings: no hace falta build command — el sitio es estático.
   `netlify.toml` ya deja configurado `publish = "."`.
3. Cada `git push` a la rama conectada dispara un deploy automático.

No hay variables de entorno necesarias para el frontend: las claves usadas
(Supabase publishable key, tokens de acceso) ya están en el código fuente por
diseño, tal como se armó el proyecto originalmente.
