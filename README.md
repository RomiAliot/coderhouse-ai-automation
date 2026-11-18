# Checkpoint 4: Sincronización del Cerebro Agéntico

**Alumna:** Romina Aliotta
**Proyecto:** Asistente virtual de Esteticabahia (centro de estética de Bahía Blanca)
**Plataforma:** n8n (local, Docker)

## Contenido del repositorio

- `checkpoint4_romina_aliotta.json`: workflow exportado desde n8n (Workflow > Download). Se puede importar desde el menú ⋯ > Import from file.
- `credentials.png`: captura de la sección Credentials de n8n con las tres conexiones activas.

## Herramientas conectadas (OAuth2)

| Rol | Herramienta | Uso |
|---|---|---|
| CRM / fuente de datos de clientes | Google Sheets | Look up de contactos por email antes de crear |
| Casilla de correo | Gmail | Trigger de correos entrantes y creación de borradores |
| Canal del equipo | Slack | Aviso al equipo en el canal #n8n-estetica |

## Flujo

1. **Gmail Trigger**: recibe el correo entrante.
2. **filtrar emails no corresponde responder**: descarta correos automáticos (Auto-reply, Out of office, Undeliverable, no-reply@, donotreply@). Los correos automáticos terminan en Stop.
3. **Limpiar datos (Edit Fields)**: extrae `email` y `nombre` del remitente, y deja solo los campos necesarios.
4. **Asegurar no esta vacio**: que le email no venga empty.
5. **AI Agent**: clasifica el correo y redacta un borrador. Devuelve JSON con `categoria`, `requiere_humano` y `borrador`.
6. **Get row(s) in sheet**: busca el email del remitente en la planilla de clientes (Look up).
7. **IF existe**: si el contacto no existe, lo crea con **Append row** en la planilla. Así se evitan duplicados.
8. **Create a draft (Gmail)**: guarda la respuesta en Borradores. **No envía nada**: la aprobación final es humana (Human-in-the-loop).
9. **Edit Fields (texto corto)**: limpia el payload para el aviso.
10. **Slack send a massage**: publica en el canal el aviso de nuevo borrador.

## Controles preventivos

- **Anti bucle de auto-respuestas**: el IF del inicio corta los correos automáticos antes de llegar al agente.
- **Duplicados**: la búsqueda por email antes de crear el contacto evita duplicados en la planilla.
- **Human-in-the-loop**: el correo nunca se envía de forma automática, solo queda como borrador.
- **Payload limpio**: el aviso a Slack solo incluye asunto y remitente, sin el cuerpo completo del correo.

## Mínimo privilegio

- Gmail: solo lectura de correos entrantes y creación de borradores.
- Google Sheets: lectura de la hoja de clientes y escritura solo para agregar contactos nuevos.
- Slack: solo `chat:write` y `channels:read`.

## Cómo probar

1. Importar el JSON en n8n.
2. Conectar las credenciales de Gmail, Google Sheets y Slack.
3. Enviar un correo de prueba desde un email que esté en la planilla y verificar el borrador en Gmail.
4. Enviar un correo automático y verificar que el flujo se corta en el IF.
