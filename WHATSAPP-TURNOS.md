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
                              └─ crear_link_de_pago[_biomagnetismo] (Mercado Pago)

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
la persona, le cuenta en qué consiste la sesión y que todas se acompañan con
terapia de sonido: frecuencias y diapasones. Si le preguntan el precio **directamente, lo dice
claro** (esconderlo genera desconfianza), siempre acompañado de lo que
incluye. Con Biomagnetismo cuenta primero por qué la sesión dura 2 horas.

Todo eso está en el *System Message* del nodo **Asistente BioBalance**:
podés cambiar el tono o las reglas escribiendo en castellano ahí mismo.

## Servicios, precios y señas

| Servicio | Duración | Precio | Seña (50%) | Horario |
|---|---|---|---|---|
| Método Códica | 1 h | $30.000 | $15.000 | Lun a vie, 9 a 15 hs |
| Masaje Kobido | 1 h | $30.000 | $15.000 | Lun a vie, 9 a 15 hs |
| Ventosas | 1 h | $30.000 | $15.000 | Lun a vie, 9 a 15 hs |
| Piedras calientes | 1 h | $30.000 | $15.000 | Lun a vie, 9 a 15 hs |
| Bio Spa Facial (Kobido + fangoterapia + biomagnetismo estético) | 2 h | $60.000 | $30.000 | Lun a vie, inicio 9 a 13 hs |
| Biomagnetismo | 2 h | $60.000 | $30.000 | Lun a vie, 18 y 20 hs |

Todas las terapias se acompañan con **terapia de sonido: frecuencias y
diapasones**, y el asistente lo menciona siempre que explica un servicio.

Hay dos herramientas de pago separadas, `crear_link_de_pago` (sesiones de
1 h) y `crear_link_de_pago_2_horas` (Biomagnetismo y Bio Spa Facial). Así
el monto sale siempre de la configuración y nunca lo elige la IA.

**Preparación de Biomagnetismo:** cuando se paga la seña de un turno de
Biomagnetismo, además de la confirmación se manda por WhatsApp un mensaje
con la preparación. El texto se escribe en el nodo **Preparación para
Biomagnetismo** del workflow de Mercado Pago. Mientras diga "COMPLETAR",
no se manda nada.

Dirección: **Juana Azurduy 2080, Santa Rosa, La Pampa**.

Precios, montos de seña, política de cancelación, dirección y nombre de la
terapeuta se completan en el nodo **Configuración del negocio**: es el único lugar
que tenés que tocar cuando cambian.

---

## Avisos para vos (estar afinadas)

Cada vez que pasa algo, te llega un aviso **por WhatsApp a tu número
personal** y **por mail** (el mail siempre, como respaldo y registro):

| Cuándo | Ejemplo |
|---|---|
| Alguien reserva (esperando seña) | 📅 Nueva reserva — Kobido — Ana — jueves 20, 10:00 hs |
| Se acredita una seña | ✅ Seña pagada ($30.000) — Biomagnetismo — turno confirmado |
| Se libera un horario sin pagar | ⌛ Se liberó un horario — ... |
| Alguien pide hablar con vos, quiere reprogramar/cancelar, cuenta algo de salud delicado o pregunta algo que el asistente no sabe | 🙋 Ana necesita que le escribas vos — motivo — su último mensaje — link al chat |

**Si tenés un solo número (el caso de Silvina):** no se puede mandar un
WhatsApp a uno mismo, así que dejá `AVISOS_WHATSAPP` **vacío** y los avisos
llegan **solo por mail**, a **biobalance.consultas@gmail.com**. Activá las
notificaciones de esa cuenta en la app de Gmail del celular para enterarte
al momento. Si con coexistencia (ver Paso 1) seguís usando la
app de WhatsApp Business, además vas a ver todas las conversaciones del
asistente en tu celular, como siempre.

**Si algún día tenés un segundo número:** cargalo en `AVISOS_WHATSAPP` y
los avisos llegan también por WhatsApp. WhatsApp solo deja que el negocio
te escriba si le escribiste en las últimas 24 h, así que mandale un "hola"
al número del asistente cada mañana: te reconoce y te contesta "✅ Avisos
activos por 24 hs".

**Cómo se presenta el asistente:** como *"la asistente virtual de
[tu nombre]"*. Si alguien pregunta si es una persona, responde con la
verdad: que es tu asistente y que vos leés todo. Nunca dice ser vos.

## Sobre clonar tu voz

Se puede (por ejemplo con ElevenLabs: subís unos minutos de audio tuyo y
el asistente responde con notas de voz con tu voz). Recomendación:
**no por ahora**.

- Una nota de voz con tu voz hace creer que sos vos respondiendo en ese
  momento. Si después la persona se entera de que era una máquina, se
  pierde la confianza, que es lo más valioso en terapias.
- Suma costo por cada audio y otra cuenta para mantener.
- Lo que más ayuda primero es lo contrario: que el asistente **entienda
  los audios que te mandan** (en Argentina se manda muchísimo audio).
  Eso se agrega transcribiendo el audio antes de pasarlo al asistente.

Si más adelante querés tu voz, el uso que mejor funciona es uno donde
todos saben que es grabado: por ejemplo un audio de bienvenida o la
explicación de cómo es una sesión de Biomagnetismo.

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
3. **Agregar tu número real**. Acá está la decisión importante, porque
   **Silvina usa un solo número para todo**:
   - **Opción recomendada: coexistencia.** Meta permite conectar a la API
     el mismo número que ya usás en la app **WhatsApp Business**, y seguir
     usando la app en el celular. Los clientes siguen escribiendo al número
     de siempre, vos ves todos los chats en tu teléfono y podés contestar
     vos cuando quieras. Requisitos: el número tiene que estar en
     **WhatsApp Business** (no en el WhatsApp común) y con la app
     actualizada. La conexión se hace con el "registro integrado"
     (*Embedded Signup*) de Meta, que normalmente ofrecen los proveedores
     oficiales de WhatsApp (BSP). Antes de elegir uno, confirmá que
     soporte coexistencia y que dé acceso a la Cloud API, porque eso es lo
     que usa n8n.
   - **Sin coexistencia**, un número conectado a la API **deja de funcionar
     en la app del celular**: perderías tu WhatsApp. En ese caso conviene
     un chip nuevo (sale muy poco) para el asistente o para vos.
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
| Gmail OAuth2 | `Gmail account` | La cuenta de Gmail desde la que salen los avisos |

Y en **Settings → Variables** (o en el entorno del proceso de n8n) las
variables nuevas de `.env.example`: `N8N_PUBLIC_URL`,
`WHATSAPP_PHONE_NUMBER_ID`, `GOOGLE_CALENDAR_ID`, `MP_ACCESS_TOKEN`,
`AVISOS_WHATSAPP` (tu número personal) y `AVISOS_EMAIL`.

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
- **Cuando Silvina contesta un chat ella misma** (desde la app, con
  coexistencia), el asistente **todavía no se entera** y podría seguir
  respondiendo en ese chat. Próximo paso pendiente: que el asistente se
  pause solo durante unas horas en un chat donde ella escribió.

## Próximos pasos sugeridos

1. **Recordatorio 24 h antes** del turno. Necesita una *plantilla* aprobada
   por Meta, porque puede caer fuera de la ventana de 24 h.
2. **Pausa automática** del asistente en un chat cuando Silvina responde.
3. **Entender audios** que mandan los clientes (transcripción).
4. **Plantilla de aviso** en Meta para que los avisos por WhatsApp no
   dependan de la ventana de 24 h.
5. **Planilla de Google Sheets** con clientes, servicios y señas, para
   llevar números del mes.
