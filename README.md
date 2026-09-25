# Panel YPF Luz — appypf.giwa-ia.com

Panel comercial del asistente de WhatsApp de YPF Luz: resumen y embudo del bot,
pipeline de oportunidades, conversaciones completas y parques/precios. Sitio
estático, mismo patrón que el panel de Grun (`../grun-panel-pages`), pero con
una **Supabase Edge Function** como API en lugar de n8n.

```
appypf.giwa-ia.com  →  GitHub Pages (este index.html)
                           ↓ POST text/plain {"k": <clave>, "dias": 7}
                      Edge Function ypf-panel (proyecto ofhiogczijhzkuqdjwvz)
                           ↓ service_role, solo lectura
                      schema ypf (contactos, mensajes, simulaciones, oportunidades, …)
```

El código de la función vive en `../YPF LUZ/supabase/functions/ypf-panel/index.ts`.

## La clave

La clave que tipea el usuario **es** la credencial de la API. Viaja en el body
(no en la URL) y la función la compara contra el secret `PANEL_CLAVE`. Una clave
mala espera 1 s antes del 401 (freno anti fuerza bruta). En este archivo no hay
ningún secreto: es HTML público.

Por defecto la clave queda en `sessionStorage`; con "Recordarme", en
`localStorage`.

Para cambiarla: Supabase → Edge Functions → Secrets → `PANEL_CLAVE`.

## CORS

La función solo devuelve `Access-Control-Allow-Origin` a
`https://appypf.giwa-ia.com` y `http://localhost:8080` (constante `ORIGENES`).
POST `text/plain` sin headers propios no dispara preflight: si se agrega un
header al `fetch`, hay que revisar el `OPTIONS` de la función.

Probar en local:

```bash
python3 -m http.server 8080 --directory .
```

## Deploy

- **Front:** `git push` a `main` → GitHub Pages. Custom domain por el archivo
  `CNAME`; DNS en Squarespace: `CNAME appypf → sechavarria-star.github.io`.
- **Función:** Management API con un token personal (`SUPABASE_ACCESS_TOKEN` en
  `../YPF LUZ/.env`, nunca en el repo):

```bash
curl -X POST "https://api.supabase.com/v1/projects/ofhiogczijhzkuqdjwvz/functions/deploy?slug=ypf-panel" \
  -H "Authorization: Bearer $SUPABASE_ACCESS_TOKEN" \
  -F 'metadata={"entrypoint_path":"index.ts","name":"ypf-panel","verify_jwt":false};type=application/json' \
  -F "file=@supabase/functions/ypf-panel/index.ts;filename=index.ts;type=application/typescript"
```

`verify_jwt` va apagado a propósito: no hay anon key en el HTML, la clave del
panel es la única puerta.

## Qué es solo lectura

El panel no escribe nada en la base. "Escribir" en una conversación abre
WhatsApp (`wa.me`); tomar la conversación desde el panel quedaría para una
etapa 2 (necesita una acción de escritura en la función).
