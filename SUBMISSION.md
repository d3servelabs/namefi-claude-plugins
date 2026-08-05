# Submitting the Namefi plugin to the community marketplace

Anthropic reviews third-party plugins before they land in the public
`claude-community` marketplace. Approved plugins are pinned to a commit SHA in
[`anthropics/claude-plugins-community`](https://github.com/anthropics/claude-plugins-community/blob/main/.claude-plugin/marketplace.json),
and their CI bumps that pin as you push new commits here. Users then install with:

```shell
/plugin marketplace add anthropics/claude-plugins-community
/plugin install namefi@claude-community
```

## Pre-submission checklist

Run from the repo root:

```bash
claude plugin validate ./plugins/namefi --strict   # review pipeline runs this same check
claude plugin validate . --strict
claude --plugin-dir ./plugins/namefi
```

Inside that session, confirm:

- [ ] `/help` → **Custom commands** lists `/namefi:domains`
- [ ] `/mcp` shows `plugin:namefi:api`, OAuth completes, and Namefi tools appear afterwards
- [ ] An end-to-end sanity task works, e.g. "check if example-namefi-test.com is available"
- [ ] Repo is **public** on GitHub and the default branch has the commit you want reviewed
- [ ] `version` in `plugins/namefi/.claude-plugin/plugin.json` is bumped if this is a re-submission

## Submit

Use the Console form (works for individual authors and orgs):
<https://platform.claude.com/plugins/submit>

Team/Enterprise orgs with directory-management access can instead use
<https://claude.ai/admin-settings/directory/submissions/plugins/new>.

### Values to enter

| Field | Value |
| --- | --- |
| Repository | `https://github.com/d3servelabs/namefi-claude-plugins` |
| Plugin name | `namefi` |
| Plugin path in repo | `plugins/namefi` |
| Display name | Namefi |
| Description | Search, register, and manage domains (and their DNS) through the Namefi API, via the hosted Namefi MCP server. |
| Category | productivity |
| Keywords | domains, dns, domain-registration, namefi, web3, mcp |
| Homepage | https://namefi.io |
| License | MIT |
| Author / contact | Namefi (D3Serve Labs) |

The form's exact fields may differ; the table is what the manifests declare, so
answers stay consistent with what the reviewer sees.

A plugin living in a subdirectory is well supported: ~400 of the ~2,300 approved
entries in the community catalog use a `git-subdir` source with a
`plugins/<name>` path, which is exactly this layout.

### Example use cases

One-liners a user types; each maps to tools the plugin exposes.

| Prompt | What it exercises |
| --- | --- |
| "Is `acme-robotics.com` available and what does it cost?" | `checkAvailability` — price, registrar, duration bounds |
| "Find me a short .com for an AI invoicing startup" | `getSuggestions` |
| "Check these five names at once: a.com, b.io, ..." | `checkBulkAvailability` |
| "Register `acme-robotics.com` for 2 years" | confirm → `registerDomain` → poll `getOrder` |
| "Buy it with USDC instead of a card" | `registerDomainX402` |
| "Just give me a cart link, I'll check out myself" | `https://namefi.io/cart/add-from-url?add_to_cart=…` |
| "What domains do I own?" | `getUserDomains` |
| "Point `acme-robotics.com` at 76.76.21.21" | `getDnsRecords` → `createDnsRecord` |
| "Move all my MX records to Google Workspace" | `getDnsRecords` → `batchUpdateDnsRecords` |
| "Turn on auto-renew for everything I own" | `getUserDomains` → `toggleAutoRenew` |
| "Park this domain and set up forwarding to my main site" | `toggleDomainParking`, `toggleForwarding` |
| "Find buyers for the domains I'm sitting on and draft outreach" | `startOutboundRun` → `listOutboundLeads` → `prepareOutboundOutreach` |

### What reviewers will look at

- The bundled MCP server (`https://api.namefi.io/mcp`) — it's a network egress point,
  so expect questions about what data leaves the machine and how auth works. Answer:
  OAuth 2.1 + PKCE with dynamic client registration, or a user-supplied `x-api-key`;
  no credentials are bundled in the plugin.
- The skill instructs Claude to confirm with the user before submitting a paid
  registration order. Keep that behavior — it's the main safety property here.

## After approval

- The public catalog syncs nightly, so there's a delay between approval and the
  plugin appearing. Check by searching for `namefi` in the
  [community catalog](https://github.com/anthropics/claude-plugins-community/blob/main/.claude-plugin/marketplace.json).
- Ship updates by bumping `version` in `plugins/namefi/.claude-plugin/plugin.json`
  and pushing. Users only receive an update when that string changes.
- The separately curated `claude-plugins-official` marketplace has no application
  process; this form does not submit to it.
