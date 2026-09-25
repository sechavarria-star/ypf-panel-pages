# ypf-panel-pages — Panel YPF Luz (appypf.giwa-ia.com)

Panel comercial del bot de WhatsApp de YPF Luz como sitio estático. API: Edge Function
`ypf-panel` (código en `../YPF LUZ/supabase/functions/ypf-panel/`).

@README.md

## Reglas

- Repo público: **nada de datos ni secretos** en este directorio. La clave vive solo en el
  secret `PANEL_CLAVE` de la función.
- Todo lo que viene de la base se escapa con `esc()` antes de ir al HTML (los mensajes los
  escribe cualquiera por WhatsApp).
- Los avisos de calidad de datos (`alertas`) los calcula la función: no hardcodearlos acá.
- Antes de publicar: extraer el `<script>` y chequear sintaxis (no hay Node en esta máquina:
  `osascript -l JavaScript` con `new Function(src)` sirve).
