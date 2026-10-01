# Clients without OAuth

Create an API key in Numina under **Integrationer → API forbindelser → Opret API-nøgle** with the
`read:ledger` scope. Keys start with `numi_` and are shown once.

## Claude Code

```bash
claude mcp add --transport http numina https://api.numina.app/mcp \
  --header "Authorization: Bearer numi_..."
```

## JSON config (Cursor, VS Code and others)

```json
{
  "mcpServers": {
    "numina": {
      "type": "http",
      "url": "https://api.numina.app/mcp",
      "headers": { "Authorization": "Bearer numi_..." }
    }
  }
}
```

Keep the key out of version control.
