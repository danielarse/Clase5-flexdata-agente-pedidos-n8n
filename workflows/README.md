# Workflows de n8n

Coloca aquí el export JSON de tu workflow (`Download` desde el editor de n8n
o `Ctrl/Cmd + A` → `Ctrl/Cmd + C` → pegar en un archivo `.json`).

Nombre sugerido: `flexdata-pedidos.json`

## Cómo exportar desde n8n

1. Abre el workflow en n8n.
2. Menú (`⋮`) → **Download**.
3. Guarda el archivo como `workflows/flexdata-pedidos.json` en este repositorio.

## Cómo importar en otra instancia

1. En n8n: **Workflows** → **Import from File**.
2. Selecciona `flexdata-pedidos.json`.
3. Vuelve a vincular las credenciales (Outlook, Cohere, Google Sheets), ya
   que estas **no** se incluyen en el export por seguridad.

> ⚠️ Verifica antes de subir el JSON que no contenga tokens, API keys ni
> IDs sensibles embebidos en los nodos.
