# MCP Client Configuration

Configure your MCP client to connect to the dockerized docling-mcp server running on port 3010 with streamable-http transport.

## Claude Code (CLI)

```bash
# Add the MCP server to Claude Code settings
claude mcp add --transport http docling http://localhost:3010/mcp
```

## Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "docling": {
      "url": "http://localhost:3010/mcp",
      "transport": "streamable-http"
    }
  }
}
```

## LM Studio

Edit `~/.lmstudio/mcp.json`:

```json
{
  "mcpServers": {
    "docling": {
      "url": "http://localhost:3010/mcp",
      "transport": "streamable-http"
    }
  }
}
```

## Generic MCP Client (Python)

```bash
pip install mcp
```

```python
from mcp.client.streamable_http import streamablehttp_client

async with streamablehttp_client("http://localhost:3010/mcp") as (read, write, _):
    # Use the client
    pass
```

## Verify Connection

First, make sure the container is running:

```bash
docker compose up -d

# Check it's healthy
curl -X POST http://localhost:3010/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

You should get a JSON response with the server's capabilities and available tools.

Server info: `docling v1.25.0`

## Available Tool Groups

| Tool Group | Description | External Dependency |
|------------|-------------|---------------------|
| `conversion` | PDF to DoclingDocument | None |
| `generation` | Document creation | None |
| `manipulation` | Document editing | None |
| `llama-index-rag` | Milvus vector store RAG | Milvus server |
| `llama-stack-rag` | Llama Stack RAG | Llama Stack server |
| `llama-stack-ie` | Llama Stack info extraction | Llama Stack server |

### Conversion Tools (6)
- `convert_document_into_docling_document` - Convert document from URL or local path
- `convert_directory_files_into_docling_document` - Convert all files from a directory
- `export_docling_document_to_markdown` - Export document to markdown format
- `save_docling_document` - Save document to disk (markdown + JSON)
- `page_thumbnail` - Generate thumbnail image for a page
- `is_document_in_local_cache` - Check if document is already converted

### Generation Tools (8)
- `create_new_docling_document` - Create new document from prompt
- `add_title_to_docling_document` - Add or update document title
- `add_section_heading_to_docling_document` - Add section heading with level
- `add_paragraph_to_docling_document` - Add paragraph text
- `open_list_in_docling_document` - Open a new list group
- `close_list_in_docling_document` - Close a list group
- `add_list_items_to_list_in_docling_document` - Add items to open list
- `add_table_in_html_format_to_docling_document` - Add HTML table

### Manipulation Tools (5)
- `get_overview_of_document_anchors` - Get document structure overview
- `search_for_text_in_document_anchors` - Search text in document
- `get_text_of_document_item_at_anchor` - Get text at specific anchor
- `update_text_of_document_item_at_anchor` - Update text at anchor
- `delete_document_items_at_anchors` - Delete items at anchors

## Enable RAG Tools

To enable RAG tools, override the command in docker-compose.yml:

```yaml
services:
  docling-mcp:
    build: .
    ports:
      - "3010:3010"
    environment:
      - MILVUS_URI=http://milvus:19530
    command: ["--transport", "streamable-http", "--host", "0.0.0.0", "--port", "3010",
              "conversion", "generation", "manipulation", "llama-index-rag"]
    depends_on:
      - milvus

  milvus:
    image: milvusdb/milvus:latest
    ports:
      - "19530:19530"
```
