# robauto-mcp

Documentation and contract for the hosted **Robauto MCP server** — a public,
stateless Streamable HTTP endpoint that gives any AI agent a brand resolution
layer: one canonical agent card per website, grounded answers and comparisons,
support/dispute/legal contact routes and policies with a source page behind every
value, agent-readiness scans, cross-brand product search, and Robot Soul
persistent memory.

```
https://mcp.robauto.ai
```

Repo: <https://github.com/robauto-ai/mcp> · **Contract of record:** [`docs/manifest.json`](docs/manifest.json) · Human documentation (explanatory only): <https://robauto.ai/mcp-server>

```bash
gh repo clone robauto-ai/mcp && cd mcp
```

Discovery: the API catalog at <https://robauto.ai/.well-known/api-catalog> (RFC 9727
linkset) names this server alongside every other Robauto API surface, and the
homepage returns matching RFC 8288 `Link` headers.

## What this repo is (and is not)

- **It is** the public contract: tool list, inputs/outputs, taxonomy, transport
  details, deprecation dates, and reviewer instructions for the *live* server.
- **It is not** the deployment source, and it is not a reference server you can
  run. The live server is a single Deno edge function generated from
  `src/lib/mcp/` in Robauto's application repo; nothing here is deployed.
- There is deliberately **no second implementation** in this repo. An
  independently written server would drift from production immediately (it
  already did once: an early scaffold checked only robots/sitemap/llms while the
  live scanner also checks structured data, meta title/description, page load
  time, security.txt, ai-plugin.json, llms-full.txt, redirect chains, mixed
  content and CSP). Docs here track the live tool surface; code lives in one
  place.

Looking for a **client**? The MIT-licensed stdio client and npm SDK live in
[`robauto-ai/dsh-growth`](https://github.com/robauto-ai/dsh-growth).

## Server facts

- **Protocol:** MCP Streamable HTTP (`2025-06-18`, also accepts `2025-03-26`, `2024-11-05`)
- **Two surfaces:**
  - `https://mcp.robauto.ai` — **no auth.** Every tool in the table below. No account, API key or OAuth handshake.
  - `https://mcp.robauto.ai/auth` — **OAuth 2.1**, dynamic client registration. Four owner-scoped tools (`whoami`, `list_my_sites`, `get_my_site_traffic`, `get_my_scan`) that read only the consenting account's own sites, traffic and scans, under row-level security. See [`docs/mcp-server.md`](docs/mcp-server.md#authentication).
- **Transport:** stateless. `POST` and `OPTIONS` only — a standalone SSE stream (`GET`) and session teardown (`DELETE`) return `405` with `Allow: POST, OPTIONS`, which is spec-correct for a stateless server.
- **Capabilities:** `{ "tools": { "listChanged": false } }`. No `resources`, `prompts` or `sampling` — the equivalent data is published over plain HTTP (`/llms.txt`, `/llms-full.txt`, `/openapi.json`, `/ai-plugin.json`, `/x402.json`, `/api/public/*`).
- **Server version:** declared in the generated Tools block below (single source: `src/lib/mcp/catalog.ts`).
- **CORS:** `*`, exposes `mcp-session-id`
- **Payments:** read tools are free. `soul_remember` costs $0.01 USDC over x402 (Base); `soul_pay` returns tiers and terms.

## Grok @Bot Skill

Robauto is also available inside Grok as a @Bot Skill — no config file, no keys:

<https://x.ai/bot/7k0TLQBu4hPI5oE3ywRHU>

Add the skill in Grok and ask it to scan a site, pull the Agent Readiness leaderboard
or read AI search data. It calls the same tools as the MCP endpoint below.

## Connect

Claude / ChatGPT / Cursor connector config:

```json
{
  "mcpServers": {
    "robauto": {
      "type": "http",
      "url": "https://mcp.robauto.ai"
    }
  }
}
```

Because the server has write tools (`submit_brand`, `report_contact_outcome`,
`register_site_for_human`, `soul_verify`, and `soul_remember`), connect it with
**read and write** access. `soul_pay` only reads pricing and payment terms.

To reach your own account's sites, traffic and scans, add the authenticated
surface as a second entry — your client will run the OAuth flow and self-register:

```json
{
  "mcpServers": {
    "robauto-account": {
      "type": "http",
      "url": "https://mcp.robauto.ai/auth"
    }
  }
}
```

## Verify with curl

Every POST must send both Accept types or a spec-compliant server answers `406`.

```bash
curl -sS https://mcp.robauto.ai \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"curl","version":"0"}}}'

curl -sS https://mcp.robauto.ai \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

A browser `GET` on `https://mcp.robauto.ai` redirects to the human docs page; the
endpoint itself only answers `POST`.

## Tools

<!-- ROBAUTO-GENERATED:MCP start -->
**Server version — `0.7.1`** · **21 active public tools** · **6 deprecated aliases** · **4 authenticated tools**.

This block is generated from the live application catalog. A build rewrites this README, the complete tool reference, manifest, server card, agent card, `ai.txt`, `llms.txt`, and `llms-full.txt` together so the docs cannot silently lag behind the code.

**Contract of record —** [`docs/manifest.json`](docs/manifest.json) · live copy: <https://robauto.ai/.well-known/mcp/server-card.json> · raw: <https://raw.githubusercontent.com/robauto-ai/mcp/main/docs/manifest.json>. Human docs (<https://robauto.ai/mcp-server>) are explanatory, not authoritative.

| Tool | Kind | Cost | What |
| --- | --- | --- | --- |
| `get_site_agent_card` | read | free | Everything one website means to an AI agent, in a single call. |
| `ask_brand` | read | free | One question, one grounded answer, with the source page behind every fact. |
| `compare_brands` | read | free | Structured side-by-side differences across 2-5 domains. |
| `get_network_data` | read | free | Leaderboards, boosts, featured brands and AI-search KPIs behind one view parameter. |
| `find_brands` | read | free | Search the brand index by name or domain. |
| `get_brand_profile` | read | free | Every published contact and policy detail for a domain, with sources. |
| `get_brand_contacts` | read | free | Contact routes only, each with source page and last-checked date. |
| `get_brand_policies` | read | free | Policy pages for a domain, with refund window where known. |
| `get_brand_agent_endpoints` | read | free | How to talk to a brand machine-to-machine: MCP, agent card, connectors, payments. |
| `report_contact_outcome` | write | free | Close the loop: tell me if a route reached the right team or failed. |
| `submit_brand` | write | free | One call adds a brand with every contact, opt-out and policy field. |
| `agent_scan` | read | free | 25 agent-readiness checks for any domain, scored 0-100, with a fix for every gap. |
| `signal_scan` | read | free | Deprecated alias of `agent_scan` (removed 2027-03-01) |
| `get_ai_search_data` | read | free | Deprecated alias of `get_network_data` (removed 2027-03-01) |
| `get_top_sites` | read | free | Deprecated alias of `get_network_data` (removed 2027-03-01) |
| `get_top_agents` | read | free | Deprecated alias of `get_network_data` (removed 2027-03-01) |
| `get_boost_feed` | read | free | Deprecated alias of `get_network_data` (removed 2027-03-01) |
| `get_featured_brands` | read | free | Deprecated alias of `get_network_data` (removed 2027-03-01) |
| `search_products` | read | free | Cross-brand product search across every brand with an approved feed. |
| `get_brand_products` | read | free | Scoped catalog for a single domain, with freshness and commerce protocols. |
| `list_learn_content` | read | free | Robauto Learn micro-courses — full lesson text, sources and canonical URLs. |
| `list_repos` | read | free | Public git repos for the MCP server and developer toolkit. |
| `register_site_for_human` | write | free | Emails a human an invite to create a Robauto account and track their site. |
| `soul_verify` | write | free | Verify a Robot Soul from a signed Ed25519 challenge; returns a 90-day scoped JWT. |
| `soul_recall` | read | free | Read an agent's persistent memory (working / episodic / semantic). |
| `soul_remember` | write | $0.01 USDC | Write a durable memory to an agent's soul (metered in USDC over x402). |
| `soul_pay` | read | x402 (USDC on Base) | Return Robot Soul tiers and x402 payment terms for USDC on Base. |

**Data policy —** Robauto never exposes user contact details, account identities, or a single site's proprietary analytics over MCP. Site- and agent-level traffic is published only where the site owner already publishes it (the public leaderboards), and everything else is returned as network aggregates.
<!-- ROBAUTO-GENERATED:MCP end -->

> The table above is regenerated from `src/lib/mcp/catalog.ts`. Hand edits between
> the `ROBAUTO-GENERATED:MCP` markers are overwritten on the next build — change
> the catalog instead.

### Start here — the four calls that answer most questions

```bash
BASE=https://mcp.robauto.ai
H=(-H 'Content-Type: application/json' -H 'Accept: application/json, text/event-stream')

# 1. Everything one site means to an agent, in one call
curl -sS "$BASE" "${H[@]}" -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"get_site_agent_card","arguments":{"domain":"compostingtechnology.com"}}}'

# 2. One grounded answer with source pages attached
curl -sS "$BASE" "${H[@]}" -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"ask_brand","arguments":{"domain":"compostingtechnology.com","question":"Who handles partnerships?"}}}'

# 3. Side-by-side differences across two to five domains
curl -sS "$BASE" "${H[@]}" -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"compare_brands","arguments":{"domains":["example.com","example.org"],"criteria":["policies","agent_readiness"]}}}'

# 4. Add or correct a brand; every value includes its same-domain source page
curl -sS "$BASE" "${H[@]}" -d '{"jsonrpc":"2.0","id":4,"method":"tools/call","params":{"name":"submit_brand","arguments":{"domain":"example.com","name":"Example","fields":{"support_email":{"value":"support@example.com","source_url":"https://example.com/contact"}},"client_key":"my-agent"}}}'
```

For a batch, pass `brands` with up to 20 objects using the same `domain`, `name`,
`category`, and `fields` shape. Submissions are queued for human review and do
not overwrite verified values.

`get_brand_agent_endpoints` returns a brand's machine-to-machine routes (MCP
manifest, A2A agent card, OpenAI connector, OpenAPI, auth and payment URLs).
Stored values never expire — every field carries `last_checked` and its source
URL so you judge freshness yourself.

`agent_scan` takes a `depth`. `depth: "full"` (the default) runs all 25
agent-readiness checks; `depth: "basics"` runs only the Signal Basics file checks
(robots.txt, sitemap, llms.txt, security.txt, HTTPS health) — what the deprecated
`signal_scan` did. One tool, one score, one vocabulary for checking a site.

Every tool returns a typed JSON object rather than JSON stringified into prose,
so clients parse results instead of scraping them. Read/write intent is published
per tool as `kind` in [`docs/manifest.json`](docs/manifest.json) and in the table
above; MCP `annotations` (`readOnlyHint`, `destructiveHint`, `openWorldHint`) are
not yet mirrored into the manifest — see [Known gaps](#known-gaps).

There are no per-brand endpoints. `https://mcp.robauto.ai` serves every brand in the index:
resolve a name with `find_brands`, then ask for that domain by name.
Directory of every server: <https://robauto.ai/mcp-directory> · create one at
<https://robauto.ai/create-mcp> · verify one at <https://robauto.ai/mcp-checker>.

## Brand resolution surfaces (HTTP)

The same index the `*_brand*` tools read is published as plain HTTP, so a crawler
needs no MCP client:

| URL | Shape |
| --- | --- |
| `https://robauto.ai/directory` | Human index of every brand |
| `https://robauto.ai/directory.json` | Machine index (`total`, `brands[]`) |
| `https://robauto.ai/brand/{domain}` | Profile page, JSON-LD `Organization` + `ContactPoint` |
| `https://robauto.ai/brand/{domain}.json` | Structured fields |
| `https://robauto.ai/brand/{domain}.md` | Markdown twin |
| `https://robauto.ai/brand/{domain}/llms.txt` | `llms.txt` block |
| `https://robauto.ai/claim/{domain}` | Brand owner claims and corrects the profile |
| `https://robauto.ai/connect` | How to connect and report a contact outcome |

Every value carries `value`, `source_url`, `verified_at`, `method`
(`crawled` / `submitted` / `verified` / `reported`), `confidence` and `status`.
Values never expire or disappear because of age. The source URL and last-checked
timestamp remain attached so an agent can judge freshness and re-check the source.
Report a route that worked or failed
with `report_contact_outcome`; three independent clients over 48 hours mark a
field disputed and queue it for human verification. Robauto never publishes
submitter emails or named individuals — role mailboxes only.

## The taxonomy is the point

Most AI-visibility tools report every non-human hit as "AI traffic." Robauto
classifies each one and only counts two classes as AI search:

| Class | Counts as AI search | Meaning |
| --- | --- | --- |
| `llm_crawler` | yes | LLM/assistant crawlers (ChatGPT, Claude, Gemini, Copilot, Meta AI, Perplexity…) |
| `ai_referral` | yes | Humans arriving from an AI answer |
| `search_bot` | no | Classic search and SEO crawlers (Googlebot, Baiduspider, Ahrefs, Amazonbot…) |
| `headless` | no | Headless browsers, screenshot and preview services |
| `other_bot` | no | Unclassified automation |

Roughly half of classified bot traffic on the network is `llm_crawler`, so a
vendor that blends the classes reports about double Robauto's number for the
same site. `get_network_data` with `view: "ai_search"` returns the split, not a
blended figure.

**Known measurement gap:** `ai_referral` reads near zero over long windows. That
is a detection limitation, not proof that AI answers send no clicks — most AI
surfaces strip or omit the referrer, and those sessions land in direct traffic.
Treat `ai_referral` as a floor, and read `llm_crawler` as the reliable signal
until referrer-independent attribution ships.

## Known gaps

Verified against a live `agent_scan` of `robauto.ai` and the live `tools/list`, then
re-checked after the source fixes below. Published here rather than quietly fixed in
the docs, because a contract that hides its own open items is not a contract.

| Gap | Evidence | Status |
| --- | --- | --- |
| **RFC 8414 issuer-suffix path returned nothing.** Base Authorization Server metadata is valid, but scanners also fetch `/.well-known/oauth-authorization-server/<issuer-path>` and a `404` there fails the check. | `https://robauto.ai/.well-known/oauth-authorization-server/auth/v1` | fixed — the issuer-suffix form for both `oauth-authorization-server` and `openid-configuration` is answered by the edge discovery worker (it cannot be a static file, since the extensionless base document occupies the same path name). Live after the next publish. |
| **`get_site_agent_card.mcp_endpoint` returned the server *card* URL.** An agent that POSTed JSON-RPC to `…/.well-known/mcp/server-card.json` got a static document. | live call, `domain: robauto.ai` | fixed — `mcp_endpoint` now carries only a callable JSON-RPC endpoint; the static card moved to its own `mcp_server_card_url` field. |
| **`agent_scan` output referenced a `force` argument** the live input schema does not accept. Arguments are `domain`, `depth`, `include_fixes`. | live `tools/list` vs. `get_site_agent_card.signal.note` | fixed — the stale string is gone from the app catalog. |
| **Manifest `inputSchema` was lossy.** Seven properties published TypeScript union/array syntax as a `type` string. | `docs/manifest.json` vs. live `tools/list` | fixed in the generator, not just the artifact — the catalog now carries JSON Schema primitives plus `enum` / `items`, and a drift test fails the build if any published property type is not a JSON Schema type. |
| **MCP `annotations` absent from the manifest.** The manifest publishes `kind` (`read`/`write`) instead of `readOnlyHint` / `destructiveHint` / `openWorldHint`. A client reading only the contract of record cannot see annotation intent. | `docs/manifest.json` | open — `kind` is accurate but is not the MCP-standard field. |
| **Robauto's own brand record was nearly empty** — products and a score, but no contacts, policies or agent endpoints, while the scan confirms a published A2A card, MCP card, OpenAPI, OAuth discovery and x402. | `get_brand_profile { "domain": "robauto.ai" }` | fixed — support route, policy links and all seven agent endpoints are now published with their source pages. |


## Live self-scan

Robauto scores its own domain with the same 25 checks it runs on anyone else.
Re-run it yourself — no account, no key:

```bash
curl -sS https://mcp.robauto.ai \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"agent_scan","arguments":{"domain":"robauto.ai"}}}'
```

As of 2026-09-06: **98 / 100**, 24 passing, 1 warning, 0 failing.
Signal Basics 6/6 · Technical Groundwork 4/4 · Agent Integration 9/10 · Agentic Commerce 5/5.
The single warning is the RFC 8414 issuer-suffix path in the table above.

## Contract stability

Aliases keep responding until their removal date. Nothing is removed without a
version bump and a `CHANGELOG.md` entry. Last verified against the live
`tools/list` on 2026-09-06: 27 tools, exactly matching
[`docs/manifest.json`](docs/manifest.json) — no drift in either direction.

| Item | Status |
| --- | --- |
| `agent_scan` | canonical scan tool |
| `signal_scan` | deprecated alias (`agent_scan` + `depth: "basics"`); removal scheduled for **2027-03-01** |
| `get_network_data` | canonical network-data tool |
| `get_ai_search_data`, `get_top_sites`, `get_top_agents`, `get_boost_feed`, `get_featured_brands` | deprecated aliases (`get_network_data` + matching `view`); removal scheduled for **2027-03-01** |
| `get_site_agent_card` | canonical per-domain lookup; supersedes chaining `get_brand_profile` + `get_brand_contacts` + `get_brand_policies` + `get_brand_products` |
| `get_brand_profile`, `get_brand_contacts`, `get_brand_policies`, `get_brand_products` | supported, not deprecated — narrower reads for callers that want one slice |
| `scan_site`, `aeo_scan` | removed |
| `signal_strength` | canonical score field (file/HTTPS level) |
| `readiness_score` | canonical score field for the 25 agent-readiness checks |
| `score`, `aeo_score` | deprecated aliases; removal scheduled for **2026-10-01** |
| `/api/soul/agents/{agent_id}` | canonical Robot Soul path |
| `/api/soul/soul/{agent_id}` | deprecated alias; removal scheduled for **2026-10-01** |
| `POST /site-mcp?domain=…` (per-domain shard) | frozen. Still answers for clients configured before consolidation; no longer advertised and gets no new tools. Point new integrations at `https://mcp.robauto.ai`. |

## Docs

- [`docs/mcp-server.md`](docs/mcp-server.md) — architecture, transport, auth, deployment
- [`docs/tools.md`](docs/tools.md) — every tool, input, output, and cost
- [`docs/review-access.md`](docs/review-access.md) — instructions for directory reviewers
- [`docs/manifest.json`](docs/manifest.json) — generated tool manifest
- [`CHANGELOG.md`](CHANGELOG.md) — version history and deprecation notices

## License

MIT — see [LICENSE](LICENSE).
