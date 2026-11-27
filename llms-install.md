# Installing Dataset Join & Merge (remote MCP server, no install needed)

Dataset Join & Merge is a hosted remote MCP server. There is nothing to download, build or run, and no API key.

Add this to the MCP settings file (in Cline: MCP Servers, Configure, then cline_mcp_settings.json):

```json
{
  "mcpServers": {
    "dataset-join-merge": {
      "type": "streamableHttp",
      "url": "https://dataset-join-merge.nerolabs.workers.dev/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Save the file. The server's tools appear straight away. To check it works, ask the assistant to list the server's tools.

Docs: https://dataset-join-merge.nerolabs.workers.dev/docs
