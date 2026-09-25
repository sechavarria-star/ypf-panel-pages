# Panel YPF Luz — appypf.giwa-ia.com

Panel comercial del asistente de WhatsApp de YPF Luz: resumen y embudo del bot,
pipeline de oportunidades, conversaciones completas y parques/precios. Sitio
estático, mismo patrón que el panel de Grun (`../grun-panel-pages`), pero con
una **Supabase Edge Function** como API en lugar de n8n.

```
appypf.giwa-ia.com  →  GitHub Pages (este index.html)
                           ↓ POST text/plain {"k": <clave>, "dias": 7}
                      Edge Function ypf-panel (proyecto ofhiogczijhzkuqdjwvz)
                           ↓ Postgres como ypf_panel_ro (solo SELECT sobre ypf, sesión read-only)
                      schema ypf (contactos, mensajes, simulaciones, oportunidades, …)
```

El código de la función vive en `../YPF LUZ/supabase/functions/ypf-panel/index.ts`.

## Acceso a la base

La función **no usa la service_role**. Se conecta como `ypf_panel_ro`, un rol que:

- solo tiene `SELECT` sobre el schema `ypf` (y una política RLS de lectura en las 11
  tablas que usa el panel; `kb_chunks` queda afuera);
- tiene la sesión en `read-only` y `statement_timeout` de 15 s;
- no puede leer `public`, `auth`, `grun` ni otros schemas, ni ejecutar las `ypf_*`.

Verificado el 24/09/2026 con pruebas reales desde la función. Receta y cómo rotar la
contraseña: `../YPF LUZ/supabase/sql/rol-ypf-panel-ro.sql`.

## La clave

La clave que tipea el usuario **es** la credencial de la API. Viaja en el body
(no en la URL) y la función la compara contra el secret `PANEL_CLAVE`. Una clave
mala espera 1 s antes del 401 (freno anti fuerza bruta). En este archivo no hay
ningún secreto: es HTML público.

Por defecto la clave queda en `sessionStorage`; con "Recordarme", en
`localStorage`.

La clave **no se guarda en ningún archivo** (ni en el Drive ni en el repo): vive
solo en el secret `PANEL_CLAVE` de la función. Rotada el 24/09/2026.

Para cambiarla: `supabase secrets set PANEL_CLAVE=<nueva> --project-ref ofhiogczijhzkuqdjwvz`
(o Supabase → Edge Functions → Secrets). Toma efecto al instante: quien tenga la
anterior guardada queda afuera al próximo refresco.

## CORS

La función solo devuelve `Access-Control-Allow-Origin` a
`https://appypf.giwa-ia.com` (constante `ORIGENES`). POST `text/plain` sin
headers propios no dispara preflight: si se agrega un header al `fetch`, hay
que revisar el `OPTIONS` de la función.

Probar en local: sumar **temporalmente** `http://localhost:8080` a `ORIGENES`,
deployar, y sacarlo al terminar.

```bash
python3 -m http.server 8080 --directory .
```

## Deploy

- **Front:** `git push` a `main` → GitHub Pages. Custom domain por el archivo
  `CNAME`; DNS en Squarespace: `CNAME appypf → sechavarria-star.github.io`.
- **Función:** con la CLI de Supabase (sesión de `supabase login`), desde `../YPF LUZ`:

```bash
supabase functions deploy ypf-panel --project-ref ofhiogczijhzkuqdjwvz --no-verify-jwt --use-api
```

Secrets de la función: `PANEL_CLAVE` (la clave del panel) y `YPF_PANEL_DB_PASSWORD`
(contraseña de `ypf_panel_ro`). Host y puerto los toma de `SUPABASE_DB_URL`, que
inyecta Supabase.

`verify_jwt` va apagado a propósito: no hay anon key en el HTML, la clave del
panel es la única puerta.

## Qué es solo lectura

El panel no escribe nada en la base. "Escribir" en una conversación abre
WhatsApp (`wa.me`); tomar la conversación desde el panel quedaría para una
etapa 2 (necesita una acción de escritura en la función).
