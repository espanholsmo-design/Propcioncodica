# Propcioncodica — VS Code + Claude Code + n8n

Automatización para publicar contenido (piezas de BioBalance) en Facebook e
Instagram usando n8n, controlado desde Claude Code en VS Code.

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

- `workflows/publicar-contenido-biobalance.json` — workflow de n8n: toma un
  `caption` + URL de imagen y publica en la Page de Facebook y en la cuenta
  de Instagram Business asociada.
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
   workflows de n8n"*, *"importá `workflows/publicar-contenido-biobalance.json`"*,
   *"ejecutá el workflow X con este caption e imagen"*, etc.

## 2. Importar el workflow a n8n

En la UI de n8n (el navegador que ya tenés abierto):

1. **Workflows → Import from File** → elegí
   `workflows/publicar-contenido-biobalance.json`.
2. Abrí el nodo **Datos del post** y cargá el `caption` y la URL pública de
   la imagen a publicar (o convertí ese nodo en un Webhook/Form Trigger si
   querés dispararlo con datos externos en vez de a mano).
3. Completá en `.env` (o como variables de entorno de n8n) los datos de
   Meta: `META_PAGE_ID`, `META_PAGE_ACCESS_TOKEN`,
   `META_IG_BUSINESS_ACCOUNT_ID`, `META_IG_ACCESS_TOKEN`. Se obtienen desde
   una app de tipo *Business* en https://developers.facebook.com con los
   permisos `pages_manage_posts` e `instagram_content_publish`.
4. Ejecutá el workflow manualmente para probar.

## 3. Flujo de trabajo día a día

- Editás el workflow como JSON en VS Code (con ayuda de Claude Code) o
  visualmente en el navegador — n8n permite ambas cosas sobre el mismo
  archivo si lo reimportás.
- Le pedís a Claude Code que genere variantes del workflow (agregar Stories,
  otro paso de validación, programar el envío, etc.).
- El posteo real siempre sale por la Graph API de Meta, nunca por un login
  automatizado del navegador.
