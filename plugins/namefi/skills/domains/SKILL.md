---
name: domains
description: Search, register, and manage domain names and their DNS through Namefi. Use when the user wants to check domain availability, buy or register a domain, transfer/renew one, list domains they own, or read and edit DNS records (A, AAAA, CNAME, MX, TXT, NS, CAA...), nameservers, parking, forwarding, or auto-renew. Also for finding outbound leads/buyers for a domain.
---

# Namefi domains

This plugin bundles the Namefi MCP server, so its tools are already available as
`mcp__plugin_namefi_api__*`. **Always prefer those tools over `curl`/REST.**

## Auth

Every Namefi call is authenticated, including availability search and DNS reads —
there is no anonymous access through MCP.

If tools are missing or return 401/unauthorized:

1. Run `/mcp` and authenticate the `plugin:namefi:api` server — it supports OAuth 2.1
   + PKCE with dynamic client registration, so no pre-registered secret is needed.
2. If OAuth is unavailable (headless, CI), tell the user to generate a key at
   <https://namefi.io/api-key> and add it as a header, e.g.
   `claude mcp add --transport http namefi https://api.namefi.io/mcp --header "x-api-key: nfk_..."`.

Don't retry a failing tool more than once before surfacing the auth step to the user.

## Registration flow

1. Check availability first — never register a name the user hasn't seen priced.
2. Confirm the exact name and duration with the user before submitting an order.
   Registration spends real money and is not reversible.
3. Register (`normalizedDomainName`, `durationInYears` 1–10, optional NFT-receiving
   wallet address).
4. Poll the order until it reaches a terminal state; report the order URL
   (`https://namefi.io/orders/<orderId>`).

### Hand off to a human instead

When the user would rather pay in the browser, or agent-side payment isn't set up,
build a cart link and stop there:

```
https://namefi.io/cart/add-from-url?add_to_cart=example.com,example2.com
```

## DNS

Read records before editing so you can show a diff of what changes. Use the batch
tools for multi-record edits rather than looping single-record calls. Supported
types: A, AAAA, CNAME, MX, TXT, NS, SOA, PTR, SRV, CAA, DS, TLSA, SSHFP, HTTPS,
SVCB, NAPTR, SPF.

## REST fallback

Only if MCP is genuinely unavailable or the user asks for raw HTTP.
Base URL `https://api.namefi.io/v-next/`, auth via `x-api-key: nfk_...` or
`Authorization: Bearer <token>`.

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/search/availability?domain=X` | Availability |
| GET | `/search/bulk-availability?domains[]=X` | Bulk availability |
| POST | `/orders/register-domain` | Register |
| POST | `/orders/register-domain/records` | Register + apply DNS |
| GET | `/orders/{orderId}` | Poll order |
| GET | `/user/domains` | Domains owned |
| GET | `/dns/records?zoneName=X` | List records |
| POST/PUT/DELETE | `/dns/record`, `/dns/records`, `/dns/records/batch` | Edit records |
| PUT | `/dns/park`, `/dns/forwarding`, `/domain-config/auto-renew` | Domain config |
| POST/GET | `/outbound/runs`, `/outbound/runs/{runId}/leads` | Lead finding |

Full reference: <https://namefi.io/llms-full.txt>
