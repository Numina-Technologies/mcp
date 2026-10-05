# Setup with an API key

For clients without OAuth, or scripts. In Numina, go to
**Integrationer → Claude & AI-assistenter → Forbind**. That creates a key
with read and draft-write access and shows ready-to-paste setup. Keys start
with `numi_` and are shown once.

## Claude Code

```bash
claude mcp add --transport http numina https://api.numina.app/mcp \
  --header "Authorization: Bearer numi_..."
```

## Claude (claude.ai / Desktop)

Settings → Connectors → Add custom connector, URL
`https://api.numina.app/mcp`, and under Advanced settings a header
`Authorization` with the value `Bearer numi_...`.

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
