Flujo implementado
``` text
Activador de Gmail
        ↓
IF - ¿Es auto-reply?
   ├─ Sí → Stop - Auto reply ignorado
   └─ No
        ↓
AI Triage Email - Groq
        ↓
Normalizar Triage Email
        ↓
Set - Limpiar payload Email
        ↓
HubSpot - Look up contacto
   ├─ Sí → HubSpot - Update contacto
   └─ No → HubSpot - Create contacto
                 ↓
          Set - Payload Draft
                 ↓
          Gmail - Create Draft HITL
                 ↓
          Set - Payload mínimo Slack
                 ↓
          Slack - Alertar Operaciones
```
Gmail
Se configuró Activador de Gmail para recibir mensajes de la casilla
de soporte.
Gmail OAuth2.
Evento: mensaje recibido.
Polling: cada minuto.
Simplificación activada.
Máximo de 10 correos por consulta.
El correo de salida se maneja exclusivamente mediante Gmail - Create
Draft HITL. No se realiza un envío automático.
Control anti auto-reply
El nodo IF - ¿Es auto-reply? se encuentra inmediatamente después del
trigger de Gmail.
Filtra:
`Auto-reply`
`Out of office`
`Undeliverable`
`no-reply@`
Los mensajes detectados se detienen mediante Stop - Auto reply
ignorado, evitando bucles infinitos de respuestas automáticas.
Triage con IA
El nodo AI Triage Email - Groq analiza los datos limpios:
From
Subject
BodyText
Utiliza una taxonomía cerrada:
`BILLING`
`TECH_SUPPORT`
`SALES`
`UNKNOWN`
También genera:
`confidence`
`risk`
`draft_body`
La salida se normaliza mediante Normalizar Triage Email.
Limpieza del payload
Set - Limpiar payload Email prepara únicamente:
`From`
`Subject`
`BodyText`
`session_id`
Esto permite trabajar con un payload controlado antes de las
integraciones externas.
HubSpot y prevención de duplicados
Se utiliza HubSpot - Look up contacto para buscar previamente el
contacto por correo electrónico.
Luego, ¿Contacto existe? determina la ruta:
Sí: `HubSpot - Update contacto` (operación Crear o actualizar un
contacto).
No: `HubSpot - Create contacto`.
La búsqueda previa evita crear contactos duplicados y previene el error
409.
Human-in-the-loop
Las dos ramas de HubSpot convergen en Set - Payload Draft, que
prepara:
destinatario;
asunto;
cuerpo.
Después se ejecuta Gmail - Create Draft HITL.
El mensaje queda en Borradores para revisión humana. El workflow no
contiene un envío automático del correo.
Slack
Antes de la mensajería se utiliza Set - Payload mínimo Slack.
El payload contiene únicamente:
cliente;
asunto;
intención;
estado del borrador.
Luego se ejecuta Slack - Alertar Operaciones con OAuth2.
Esto evita trasladar información innecesaria o payloads pesados al
canal.
Seguridad y OAuth2
Se utilizaron credenciales OAuth2 para:
Gmail;
HubSpot;
Slack.
Cada integración se utiliza para las operaciones necesarias dentro de su
función en el workflow.
Continuidad con Módulo 3
El workflow fue construido a partir del proyecto del Módulo 3.
La versión de trabajo de este checkpoint es Checkpoint 4 -
Integraciones y el workflow original del Módulo 3 se conservó por
separado.
Entrega
Archivo exportado desde n8n:
`checkpoint4_Jesica_Corral.json`
El JSON se exporta desde Workflow → Download y se publica en un
repositorio público de GitHub para que pueda importarse nuevamente en
n8n.
Resultado
El workflow procesa una consulta recibida por Gmail, filtra mensajes
automáticos, clasifica el caso con IA, limpia y normaliza los datos,
consulta HubSpot para evitar duplicados, actualiza o crea el contacto,
genera un borrador para revisión humana y finalmente notifica al equipo
de operaciones mediante Slack.
