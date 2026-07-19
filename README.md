# Ensayo Vivo — página de ensayo compartido

Página pública (sólo lectura) para escuchar un ensayo compartido desde la app
**Ensayo Vivo**, sin necesidad de tener cuenta.

Es **un solo archivo estático** (`index.html`, sin build ni dependencias): se
abre en cualquier navegador y se publica copiándolo a cualquier hosting.

## Cómo funciona

1. La app genera un link con un token de 64 hex (tabla `share_links`).
2. El link apunta a esta página con el token **en el fragmento**:
   `https://<host>/#<token>`. Va en el `#` y no en el query a propósito: así el
   token no llega al servidor del hosting, no queda en sus logs ni viaja en el
   `Referer`.
3. La página le pide los datos a la Edge Function `share` de Supabase
   (`?format=json`), que valida el token (vencimiento y revocación) con
   service-role y responde con los metadatos + una signed URL fresca del audio.

Muestra: reproductor, waveform, notas ancladas a la timeline, clips (que se
reproducen como rango del mismo audio) y canciones tocadas.

**Sólo lectura**: no permite crear ni editar nada, y no expone los nombres de
los autores de las notas.

## Publicar

Cualquier hosting estático sirve. Con GitHub Pages: Settings → Pages → Deploy
from a branch → `main` / `root`.

## Configuración

La única constante a tocar está al principio del `<script>`:

```js
var SUPABASE_URL = 'https://ykgbqhhnrewgmujauztt.supabase.co';
```

No hay claves acá: la URL del proyecto es pública, y el acceso lo da el token
del link (que la Edge Function valida).

## Nota sobre la Edge Function

La página existe porque las Edge Functions en el dominio compartido
`*.supabase.co` **no pueden servir HTML renderizable** (el gateway fuerza
`text/plain` + CSP `sandbox` como anti-phishing). Por eso la función devuelve
JSON y la página se hostea aparte. El código de la función vive en el repo de la
app, en `supabase/functions/share/`.
