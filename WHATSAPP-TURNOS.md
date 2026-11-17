# Asistente de turnos por WhatsApp — BioBalance

Un asistente que contesta por WhatsApp mientras la persona pregunta, le
explica los servicios, le ofrece horarios libres, bloquea el turno y le
manda el link de Mercado Pago para la seña. Cuando la seña se acredita, el
turno se confirma solo. Si no paga a tiempo, el horario se libera.

```
Cliente escribe ─► n8n ─► Claude (asistente) ─► responde por WhatsApp
                              │
                              ├─ ver_agenda        (Google Calendar)
                              ├─ reservar_turno    (evento "PENDIENTE SEÑA")
                              └─ crear_link_de_pago (Mercado Pago)

Mercado Pago avisa el pago ─► n8n ─► evento "CONFIRMADO" ─► WhatsApp "¡Listo!"
Cada 15 min ─► señas no pagadas en 70 min ─► se borra el evento ─► aviso
```

## Los 3 workflows

| Archivo | Qué hace |
|---|---|
| `workflows/whatsapp-asistente-turnos-biobalance.json` | Recibe mensajes, conversa, consulta la agenda, reserva y genera el link de seña |
| `workflows/mercadopago-confirmar-sena-biobalance.json` | Recibe el aviso de Mercado Pago, verifica el pago **consultándolo a la API** (así nadie puede confirmar un turno con un aviso falso) y confirma |
| `workflows/liberar-senas-vencidas-biobalance.json` | Cada 15 minutos libera los turnos pendientes cuya seña no se pagó |

## Cómo se comporta con el precio

El asistente **no arranca hablando de plata**: primero pregunta qué le pasa a
la persona, le cuenta en qué consiste la sesión y que todas incluyen terapia
de sonido con diapasones. Si le preguntan el precio **directamente, lo dice
claro** (esconderlo genera desconfianza), siempre acompañado de lo que
incluye. Con Biomagnetismo cuenta primero por qué la sesión dura 2 horas.

Todo eso está en el *System Message* del nodo **Asistente BioBalance**:
podés cambiar el tono o las reglas escribiendo en castellano ahí mismo.

## Horarios configurados

- **Lunes a viernes, 9 a 15 hs**, sesiones de 1 hora: Método Códica, Masaje
  Kobido, Ventosas y Piedras calientes.
- **Lunes a viernes, 18 a 22 hs**, sesiones de 2 horas (18 y 20 hs):
  Biomagnetismo.
- Todas con terapia de sonido con diapasones.

Precios, seña, política de cancelación, dirección y nombre de la terapeuta
se completan en el nodo **Configuración del negocio**: es el único lugar
que tenés que tocar cuando cambian.

---

## Paso 1 — Conseguir la API de WhatsApp (WhatsApp Cloud API)

La API **no está en la app de WhatsApp**: está en el panel de developers
de Meta, el mismo que ya usaste para Instagram/Facebook.

1. Entrá a **https://developers.facebook.com** → **Mis apps** → **Crear
   app**. En "caso de uso" elegí **"Conectar con clientes a través de
   WhatsApp"** y asociala a tu **portfolio comercial** (Business Manager) de
   BioBalance.
2. Dentro de la app, menú izquierdo **WhatsApp → Configuración de la API**
   (*API Setup*). Ahí Meta te da un **número de prueba**, un **token
   temporal** y el **Phone number ID**. Con eso ya podés probar mandándote
   mensajes a tu propio celular (agregalo como destinatario de prueba).
3. **Agregar tu número real**: en esa misma pantalla, **Agregar número de
   teléfono**. Ojo:
   - Un número registrado en la API **deja de funcionar en la app común de
     WhatsApp** del celular. Lo más simple es usar un número nuevo (un chip
     aparte) para el asistente.
   - Si querés usar el mismo número que ya usás en **WhatsApp Business**
     (la app), Meta tiene una opción de "coexistencia" que permite seguir
     usando la app y la API a la vez. Se activa desde algunos proveedores
     oficiales al conectar el número; si te interesa, lo vemos aparte.
4. **Token permanente** (el temporal dura 24 h): en
   **business.facebook.com → Configuración → Usuarios del sistema** creá
   un usuario del sistema administrador, asignale la app y la cuenta de
   WhatsApp, y generá un token con los permisos `whatsapp_business_messaging`
   y `whatsapp_business_management`.
5. **Verificar el negocio** (Configuración → Centro de seguridad) para
   salir de los límites de prueba.
6. **Método de pago** en la cuenta de WhatsApp: las conversaciones que
   inicia el cliente son gratis dentro de las 24 h; lo que inicia el
   negocio (ej. recordatorios fuera de esas 24 h) se paga por mensaje.

## Paso 2 — Credenciales en n8n

En n8n → **Credentials → New**:

| Credencial | Nombre que esperan los workflows | Qué cargar |
|---|---|---|
| WhatsApp API | `WhatsApp account` | Token permanente + WhatsApp Business Account ID |
| WhatsApp OAuth API (para el Trigger) | `WhatsApp OAuth account` | App ID y App Secret (Meta → tu app → Configuración → Básica) |
| Google Calendar OAuth2 API | `Google Calendar account` | Login con la cuenta de Google de BioBalance |
| Anthropic | `Anthropic account` | API key de https://console.anthropic.com |

Y en **Settings → Variables** (o en el entorno del proceso de n8n) las
variables nuevas de `.env.example`: `N8N_PUBLIC_URL`,
`WHATSAPP_PHONE_NUMBER_ID`, `GOOGLE_CALENDAR_ID`, `MP_ACCESS_TOKEN`.

> n8n tiene que ser accesible desde internet con **https** para que Meta y
> Mercado Pago le puedan avisar. Si hoy lo corrés en `localhost`, la forma
> simple es subirlo a n8n Cloud o a Railway.

## Paso 3 — Google Calendar

1. Creá un calendario nuevo, ej. **"Turnos BioBalance"**, y copiá su ID
   (Configuración del calendario → Integrar el calendario).
2. Si un día no atendés (feriado, vacaciones), creá un evento que ocupe esas
   horas en ese calendario: el asistente lo ve como ocupado.
3. Los turnos aparecen como `PENDIENTE SEÑA - servicio - nombre` y pasan a
   `CONFIRMADO - ...` (en verde) cuando se paga.

## Paso 4 — Mercado Pago

1. **https://www.mercadopago.com.ar/developers** → **Tus integraciones →
   Crear aplicación** (tipo: Checkout Pro / pagos online).
2. Copiá el **Access Token**: primero el de **prueba** para testear con
   usuarios de prueba, después el de **producción**.
3. No hace falta configurar el webhook en el panel: cada link de pago ya
   lleva la dirección de aviso (`notification_url`).

## Paso 5 — Importar y activar

1. n8n → **Import from File** → los 3 JSON de `workflows/`.
2. En **Configuración del negocio** completá precios, seña, dirección y
   nombre.
3. Activá los 3 workflows. Al activar el de WhatsApp, n8n registra el
   webhook en Meta automáticamente.
4. Probá desde tu celular: "Hola, ¿qué es el biomagnetismo?" → pedí un turno
   → pagá la seña con un usuario de prueba de Mercado Pago → tiene que
   llegarte la confirmación.

---

## Costos aproximados

- **WhatsApp**: responder a quien te escribe es gratis (ventana de 24 h).
- **Claude**: el asistente usa `claude-opus-5-5` con esfuerzo bajo. Cada
  mensaje cuesta una fracción de centavo de dólar, y con el caché de prompt
  activado el texto fijo de las instrucciones sale más barato. Si querés
  bajar el costo, podés cambiar el modelo del nodo **Claude** a
  `claude-haiku-5-5`. Conviene probar unas conversaciones antes, porque
  maneja peor la agenda.
- **Mercado Pago**: la comisión habitual por cobro.

## Cosas a tener en cuenta

- **Memoria de la conversación**: el nodo *Simple Memory* guarda las
  últimas 20 interacciones por teléfono **en la memoria de n8n**. Si n8n se
  reinicia, el asistente "olvida" las charlas en curso (la agenda no se
  pierde). Para producción conviene cambiarlo por *Postgres Chat Memory* o
  *Redis Chat Memory*.
- **Pago tardío**: si alguien paga después de que se liberó el horario, el
  workflow de confirmación va a fallar al buscar el turno. Lo vas a ver en
  *Executions* de n8n: en ese caso escribile a mano y reprogramá.
- **Números de Argentina**: WhatsApp entrega los números como `549...`. Si
  en el número de prueba de Meta los envíos fallan, agregá el destinatario
  de prueba exactamente como aparece en el mensaje recibido.
- **Salud**: el asistente tiene la instrucción de no prometer curas ni dar
  indicaciones médicas, y de derivar a la terapeuta los casos delicados
  (embarazo, marcapasos, enfermedades).
- **Hablar con una persona**: si el cliente lo pide, el asistente avisa que
  le vas a escribir. Por ahora **no te notifica a vos**: revisá los chats o
  agregá un nodo que te mande un aviso (ej. un WhatsApp a tu número
  personal o un mail).

## Próximos pasos sugeridos

1. **Recordatorio 24 h antes** del turno. Necesita una *plantilla* aprobada
   por Meta, porque puede caer fuera de la ventana de 24 h.
2. **Aviso a la terapeuta** cuando alguien pide hablar con una persona o
   cuando entra una seña.
3. **Planilla de Google Sheets** con clientes, servicios y señas, para
   llevar números del mes.
