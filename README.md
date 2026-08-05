# Namefi plugins for Claude Code

A Claude Code plugin marketplace hosting the **Namefi** plugin: search, register,
and manage domains and their DNS from Claude Code.

## Install

```shell
/plugin marketplace add d3servelabs/namefi-claude-plugins
/plugin install namefi@namefi
```

Or from namefi.io, once `hosted/.claude-plugin/marketplace.json` is deployed there:

```shell
/plugin marketplace add https://namefi.io/.claude-plugin/marketplace.json
/plugin install namefi@namefi
```

Both register a marketplace named `namefi`, and a user can hold only one
marketplace per name — adding the second replaces the first. Same plugin either
way, so this is harmless; just don't expect both to coexist.

## What's in the plugin

| Component | What it does |
| --- | --- |
| MCP server `plugin:namefi:api` | The hosted Namefi MCP server (`https://api.namefi.io/mcp`) — availability search, registration + order polling, DNS records, domain config, outbound lead-finding |
| Skill `/namefi:domains` | Teaches Claude the MCP-first flow: confirm before spending, register → poll order, cart hand-off for human checkout, and the REST fallback table |

## Authentication

The MCP server requires auth on **every** call, including availability search.

- **OAuth (default, no setup):** run `/mcp`, pick `plugin:namefi:api`, authenticate in
  the browser. Until you do, the server loads but exposes no tools. The server supports OAuth 2.1 + PKCE with dynamic client registration.
- **API key (headless / CI):** generate one at <https://namefi.io/api-key> and add
  the server yourself with a header:

  ```bash
  claude mcp add --transport http namefi https://api.namefi.io/mcp \
    --header "x-api-key: nfk_YOUR_KEY"
  ```

  The bundled `.mcp.json` deliberately ships **without** an API-key header — an
  unset `${VAR}` is passed through literally by Claude Code, which would break the
  server for everyone who hasn't exported the key.

## Local development

```bash
claude plugin validate ./plugins/namefi --strict   # manifests only — not SKILL.md or .mcp.json
claude plugin validate . --strict
claude --plugin-dir ./plugins/namefi
```

`validate` only checks the manifests. Before submitting, load the plugin and check
that `/namefi:domains` is listed under `/help` → Custom commands, then authenticate
`plugin:namefi:api` in `/mcp` and confirm its tools actually appear.

Run `/reload-plugins` after edits instead of restarting.

## Layout

```
.claude-plugin/marketplace.json         # git-based marketplace (relative source)
hosted/.claude-plugin/marketplace.json  # deploy to namefi.io (git-subdir source)
plugins/namefi/
├── .claude-plugin/plugin.json          # plugin manifest
├── .mcp.json                           # bundled Namefi MCP server
└── skills/domains/SKILL.md             # /namefi:domains
SUBMISSION.md                           # community-marketplace submission runbook
```

### Hosting the marketplace on namefi.io

Serve `hosted/.claude-plugin/marketplace.json` at
`https://namefi.io/.claude-plugin/marketplace.json` (same drop-a-static-file move
as `.well-known/mcp/servers.json`). Content-type `application/json`.

A URL-based marketplace downloads **only** that one JSON file — no repo clone — so
its plugin entry can't use a relative `./plugins/namefi` path. That's why the
hosted copy uses a `git-subdir` source pointing back at this repo, while the root
copy (used by `/plugin marketplace add d3servelabs/namefi-claude-plugins`) uses the
relative path. Keep the two in sync when plugin metadata changes.

## Submitting to the community marketplace

See [SUBMISSION.md](./SUBMISSION.md) for the checklist, the form at
<https://platform.claude.com/plugins/submit>, and the exact values to enter.

## License

MIT
