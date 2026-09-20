# Propcioncodica — VS Code + Claude Code + n8n

Automatización para publicar videos virales de BioBalance en Instagram Reels,
Facebook Reels, TikTok y YouTube Shorts usando n8n, controlado desde Claude
Code en VS Code.

## Por qué así (y no "controlando el navegador")

Automatizar el *login* del navegador para publicar en Facebook/Instagram
viola los Términos de Servicio de Meta y es frágil (rompe con cada cambio de
la UI, dispara detección anti-bot, puede terminar en el baneo de la cuenta).
En su lugar, este workflow usa la **Graph API oficial de Meta**, que es la
forma soportada de publicar desde una app/automatización propia. El
"navegador" que usás sos vos: la interfaz web de n8n (donde ya estás
logueado) y, una vez, el panel de developers de Meta para generar el token.

Si en el futuro necesitás automatizar algo que Meta realmente no expone por
API, se puede agregar un nodo de automatización de navegador (Puppeteer/
Playwright) como paso aparte — pero no para el login de la cuenta.

## Piezas

- `workflows/publicar-contenido-biobalance.json` — workflow simple para
  imágenes: `caption` + URL de imagen → Facebook Page + Instagram feed.
- `workflows/publicar-video-viral-biobalance.json` — workflow para **video**:
  toma `videoUrl` + `caption` + `title` y publica en paralelo como Instagram
  Reel, Facebook Reel, TikTok y YouTube Short.
- `.mcp.json` — conecta Claude Code (en VS Code) con tu instancia de n8n vía
  MCP, para poder crear/editar/disparar workflows desde el chat.
- `.env.example` — variables que necesitás completar.

## 1. Conectar Claude Code a tu n8n

Ya tenés n8n abierto en el navegador — solo hace falta darle a Claude Code
la URL y una API key para que pueda hablarle por atrás:

1. En n8n: **Settings → n8n API → Create an API key**.
2. Copiá `.env.example` a `.env` y completá `N8N_API_URL` (la URL que tenés
   abierta, ej. `http://localhost:5678`) y `N8N_API_KEY`.
3. En VS Code, con la extensión de Claude Code instalada, abrí este repo.
   Claude Code lee `.mcp.json` automáticamente y te va a pedir aprobar el
   servidor MCP `n8n` la primera vez — aceptalo.
4. A partir de ahí podés pedirle a Claude Code, en el chat: *"listá mis
   workflows de n8n"*, *"importá
   `workflows/publicar-video-viral-biobalance.json`"*, *"ejecutá el workflow
   de video con esta URL y este caption"*, etc.

## 2. Importar el workflow de video a n8n

En la UI de n8n (el navegador que ya tenés abierto):

1. **Workflows → Import from File** → elegí
   `workflows/publicar-video-viral-biobalance.json`.
2. Abrí el nodo **Datos del video** y cargá `videoUrl` (URL pública del
   .mp4), `caption` y `title` (o convertí ese nodo en un Webhook/Form Trigger
   si querés dispararlo con datos externos en vez de a mano).
3. Configurá las variables de entorno de n8n con los datos de `.env`
   (`Settings → Variables`, o pasándolas al proceso de n8n si lo corrés vos).
4. Ejecutá el workflow manualmente para probar — revisá cada rama por
   separado la primera vez, porque cada plataforma tiene su propio setup
   (abajo). Los nodos de subida de binario (Facebook, YouTube) y los de
   espera de procesamiento (Instagram) son los que más conviene mirar en el
   editor antes de confiar en la ejecución automática.

### Instagram Reels / Facebook Reels

Misma app de Meta que ya tenías para fotos, pero necesitás además el permiso
`instagram_content_publish` (ya lo pedís) y que la cuenta de IG sea Business/
Creator vinculada a la Page. El nodo "Esperar procesamiento IG" usa una
espera fija de 30s; si el video es largo, Meta puede tardar más — conviene
reemplazarlo por un loop que consulte `status_code` hasta `FINISHED`.

### TikTok

1. Creá una app en https://developers.tiktok.com y pedí el scope
   `video.publish` (Content Posting API — Direct Post).
2. Mientras la app no esté auditada/aprobada, TikTok solo permite publicar
   con `privacy_level: SELF_ONLY` (por eso el workflow lo trae así por
   defecto) o a cuentas de test que vos mismo agregues como developer.
3. Hacé el login OAuth una vez para obtener `access_token` y
   `refresh_token`, y cargá el `access_token` en `.env`. Como expira cada
   ~24hs, vas a necesitar un workflow aparte (o un nodo Cron en este mismo)
   que lo refresque con el `refresh_token` — no está incluido todavía.

### YouTube Shorts

1. En Google Cloud Console: creá un proyecto, habilitá **YouTube Data API
   v3** y creá credenciales OAuth 2.0 (tipo "Aplicación web" o "Desktop").
2. En n8n: **Credentials → New → YouTube OAuth2 API**, nombrala
   `YouTube account` (el workflow ya referencia ese nombre) y completá el
   flujo de login con la cuenta de YouTube de BioBalance.
3. Para que YouTube lo trate como Short, el video debe ser vertical
   (9:16) y de hasta 3 minutos, y el título/descr incluir `#Shorts` (el
   nodo ya lo agrega al título).

## 3. Flujo de trabajo día a día

- Editás el workflow como JSON en VS Code (con ayuda de Claude Code) o
  visualmente en el navegador — n8n permite ambas cosas sobre el mismo
  archivo si lo reimportás.
- Le pedís a Claude Code que genere variantes del workflow (agregar Stories,
  polling real de estado en vez de espera fija, refresco de token de
  TikTok, programar el envío con un Cron Trigger, etc.).
- El posteo real siempre sale por las APIs oficiales de cada plataforma
  (Graph API de Meta, TikTok Content Posting API, YouTube Data API), nunca
  por un login automatizado del navegador.
