# Lamina — Claude plugin

Lamina is a creative director for AI **image, video, audio, and narrated multi-shot video**.
You describe the outcome; Lamina selects and stitches the right models/apps into a costed,
approved, budget-bounded plan — then executes it.

This repository is both a **Claude plugin marketplace** and the **Lamina plugin** it lists. The
plugin ships:

- the **`lamina` skill** (`skills/lamina/SKILL.md`) — usage guidance for the lifecycle, and
- the **hosted OAuth MCP server** (`https://app.uselamina.ai/mcp/agent/v2`) — no local install,
  no API keys; you authorize once via OAuth on first connect.

No product code is distributed here — the MCP is hosted. Only the manifests and the skill
markdown live in this repo.

## Install

From Claude Code:

```
/plugin marketplace add uselamina/plugin
/plugin install lamina@lamina
```

On first use, Claude Code opens the OAuth flow to connect the hosted Lamina MCP. The seven
task-level tools (`lamina_plan`, `lamina_execute`, `lamina_status`, `lamina_cancel`,
`lamina_upload_asset`, `lamina_brand_context`, `lamina_credits`) then become available, along
with the `lamina` skill's usage guidance.

Update later with:

```
/plugin marketplace update lamina
```

## Links

- Product: https://uselamina.ai
- Developers: https://uselamina.ai/agents

## License

MIT — see [LICENSE](LICENSE).
