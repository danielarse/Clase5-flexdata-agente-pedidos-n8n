# 🏗️ Arquitectura técnica del workflow

Detalle nodo por nodo del flujo de automatización de pedidos de FlexData.

---

## 0. Vista general del canvas (n8n)

![Vista general del workflow](capturas/00-vista-general-workflow.png)

El workflow completo está compuesto por 3 nodos principales conectados en
secuencia, más dos sub-nodos que alimentan al AI Agent:

1. **Microsoft Outlook Trigger** — dispara el flujo al recibir un correo.
2. **AI Agent** — analiza la solicitud y genera la respuesta (usa como
   *Chat Model* el nodo **Cohere Chat Model** y como *Tool* el nodo
   **Pedidos**, que escribe en Google Sheets con `append: sheet`).
3. **Send a message** — nodo de Outlook que envía la respuesta al cliente.

> 💡 A diferencia de la descripción inicial (flujo 100% lineal), en la
> implementación final **Pedidos** (Google Sheets) está conectado al AI
> Agent como **Tool**, no como un nodo posterior independiente. Esto le
> permite al agente decidir cuándo registrar el pedido como parte de su
> propio razonamiento, en lugar de que el registro ocurra siempre después
> de generar la respuesta.

Ver captura: [`00-vista-general-workflow.png`](capturas/00-vista-general-workflow.png)

---

## 1. Recepción de Correo Corporativo

**Tipo de nodo:** Microsoft Outlook Trigger

- **Credencial:** cuenta corporativa `daniel.estudio.alura.2026@outlook.com`
- **Poll Times:** Every Minute
- **Trigger On:** Message Received
- **Output:** Simplified

**Salida de ejemplo:**

```json
{
  "id": "AQMkADAwATM0MDAAMi1hYzI5LWZjM2EtMDACLTAwCgBGAAAD...",
  "conversationId": "AQQkADAwATM0MDAAMi1hYzI5LWZjM2EtMDACLTAwCgAQALMo8kg3SGlBrOWsiO7qosw=",
  "subject": "Pedido notebooks",
  "bodyPreview": "Hola, necesito unas 25 notebooks para mis estudiantes, pero no sé qué modelos elegir, requiero mayor información de precio y calidad.",
  "from": "daniel.estudio.alura.2026@gmail.com",
  "to": ["daniel.estudio.alura.2026@outlook.com"],
  "categories": [],
  "hasAttachments": false
}
```

Ver captura: [`01-recepcion-correo-corporativo.png`](capturas/01-recepcion-correo-corporativo.png)

---

## 2. Análisis y Extracción Inteligente de Solicitudes

**Tipo de nodo:** AI Agent

- **Source for Prompt (User Message):** Define below
- **Prompt:** contiene el rol del agente *Pablo*, vendedor de FlexData, encargado de atender al cliente, completar la planilla de pedidos y responder con tono amable, amigable y guiándolo en su consulta.
- **Chat Model:** conectado al nodo Cohere (paso 3)
- **Salida:** HTML con la respuesta personalizada para el cliente, incluyendo catálogo de modelos, precios y siguientes pasos.

Prompt completo disponible en [`src/prompts/agente-ventas-pablo.md`](../src/prompts/agente-ventas-pablo.md).

Ver captura: [`02-analisis-extraccion-inteligente.png`](capturas/02-analisis-extraccion-inteligente.png)

---

## 3. Motor de Comprensión de Solicitudes

**Tipo de nodo:** Cohere Chat Model

- **Credencial:** Cohere account
- **Modelo:** `command-a-03-2025`
- Conectado como *Chat Model* del AI Agent del paso 2.

Ver captura: [`03-motor-comprension-solicitudes.png`](capturas/03-motor-comprension-solicitudes.png)

---

## 4. Registro de Pedidos (Google Sheets)

**Tipo de nodo:** Google Sheets

- **Credencial:** Google Sheets account
- **Resource:** Sheet Within Document
- **Operation:** Append Row
- **Document:** Planilla de Pedidos de FlexData
- **Sheet:** Pedidos
- **Mapping Column Mode:** Map Each Column Manually

**Columnas mapeadas:**

| Columna | Valor / fuente |
|---|---|
| Fecha | `{{ $now.format('dd-MM-yyyy') }}` |
| Canal | Definido automáticamente por el modelo de IA (Gmail, Outlook, LinkedIn, Instagram, WhatsApp, etc.) |
| Cliente | `{{ $json.from }}` |
| Asunto | `{{ $json.subject }}` |
| Mensaje | `{{ $json.bodyPreview }}` |

Ver capturas: [`04a-registro-pedidos-sheets.png`](capturas/04a-registro-pedidos-sheets.png) y [`04b-registro-pedidos-sheets-mapeo.png`](capturas/04b-registro-pedidos-sheets-mapeo.png)

---

## 5. Respuesta Automática al Cliente

**Tipo de nodo:** Microsoft Outlook

- **Resource:** Message
- **Operation:** Send
- **To:** `{{ $('Recepción de Correo Corporativo').item.json.from }}`
- **Subject:** `{{ $('Recepción de Correo Corporativo').item.json.subject }}`
- **Message:** `{{ $json.output }}` (HTML generado por el agente en el paso 2)
- **Message Type:** HTML

Ver captura: [`05-respuesta-automatica-cliente.png`](capturas/05-respuesta-automatica-cliente.png)
