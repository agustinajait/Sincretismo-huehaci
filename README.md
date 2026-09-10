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
  - **Organización** ("Hacer tu diagnóstico"): abre un modal con dos pestañas.
    - *Ya tengo código*: valida el código contra Supabase y redirige a
      `diagnostico.html?org=N&token=X`.
    - *Primera vez*: la organización se registra sola — elige su nombre y su
      propio código de acceso, sin que nadie tenga que pasarle un link. El
      código queda guardado como `access_token` de su fila en Supabase, así
      que sirve también para volver a entrar más adelante desde la pestaña
      *Ya tengo código*.
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
token de administrador está hardcodeado en el código; los tokens por
organización se generan al crear cada fila (desde el panel de admin, o desde
el registro propio en `index.html`) y quedan guardados en `access_token`.

La tabla `sincretismo_nomina` tiene RLS habilitado con políticas públicas de
`SELECT`/`INSERT`/`UPDATE` (sin auth de por medio) — el acceso no lo resuelve
Postgres sino el `access_token` que valida el frontend. Es lo que permite que
una organización se registre sola desde `index.html` sin pasar por el panel
de admin.

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
