# Lamina — OpenAI and Claude plugin

Lamina is a creative director for AI **image, video, audio, and narrated multi-shot video**.
You describe the outcome; Lamina selects and stitches the right models/apps into a costed,
approved, budget-bounded plan — then executes it.

This repository packages Lamina for both **OpenAI** and **Claude**. It is also the Claude plugin
marketplace that lists Lamina. The plugin ships:

- the **`lamina` skill** (`skills/lamina/SKILL.md`) — usage guidance for the lifecycle, and
- the **curated OpenAI OAuth MCP server** (`https://app.uselamina.ai/mcp/agent/openai`), and
- the backwards-compatible **Claude/v2 OAuth MCP server**
  (`https://app.uselamina.ai/mcp/agent/v2`).

Both servers are hosted: there is no local runtime or API key to install. Users authorize once
with Lamina on first connect.

No product code is distributed here — the MCP is hosted. Only the manifests and the skill
markdown live in this repo.

## OpenAI package

The OpenAI bundle is declared by `.codex-plugin/plugin.json`, connects through `.mcp.json`, and
uses the same `skills/lamina` source as Claude. It exposes exactly nine task-level tools for
planning, choosing, approving, executing, monitoring, steering, uploading assets, reading brand
context, and checking credits. Once the directory listing is approved and published, install
Lamina from the OpenAI Plugins Directory.

## Claude install

From Claude Code:

```
/plugin marketplace add uselamina/plugin
/plugin install lamina@lamina
```

On first use, Claude Code opens the OAuth flow to connect the hosted Lamina MCP. The nine
task-level tools (`lamina_plan`, `lamina_choose`, `lamina_execute`, `lamina_status`,
`lamina_cancel`, `lamina_steer`, `lamina_upload_asset`, `lamina_brand_context`, and
`lamina_credits`) then become available, along with the `lamina` skill's usage guidance.

Update later with:

```
/plugin marketplace update lamina
```

## Links

- Product: https://uselamina.ai
- Developers: https://uselamina.ai/agents
- Support: https://uselamina.ai/support
- Privacy: https://uselamina.ai/privacy-policy
- Terms: https://uselamina.ai/terms-of-service

## License

MIT — see [LICENSE](LICENSE).
