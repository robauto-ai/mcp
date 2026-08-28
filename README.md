# robauto-mcp

Documentation and contract for the hosted **Robauto MCP server** — a public,
stateless Streamable HTTP endpoint that gives any AI agent Robauto's tools:
Signal Strength scans, network AI-search data, human invites, and Robot Soul
persistent memory.

```
https://mcp.robauto.ai
```

Repo: <https://github.com/robauto-ai/mcp> · Human documentation: <https://robauto.ai/mcp-server>

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
- **Auth:** none. No account, API key or OAuth handshake.
- **Transport:** stateless. `POST` and `OPTIONS` only — there is no standalone SSE stream (`GET`) and no session (`DELETE`).
- **Server version:** `0.3.0`
- **CORS:** `*`, exposes `mcp-session-id`

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

Because the server has write tools (`register_site_for_human`, `soul_remember`,
`soul_pay`), connect it with **read and write** access.

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
| Tool | Kind | Cost | What |
| --- | --- | --- | --- |
| `agent_scan` | read | free | 25 agent-readiness checks for any domain, scored 0-100, with a fix for every gap. |
| `signal_scan` | read | free | Signal Strength (0-100) + per-file findings for any public domain. |
| `scan_site` | read | free | Deprecated alias of `signal_scan` (removed 2026-10-01) |
| `aeo_scan` | read | free | Deprecated alias of `signal_scan` (removed 2026-10-01) |
| `get_ai_search_data` | read | free | Aggregate AI-search KPIs and bot taxonomy across the Robauto network. |
| `get_top_sites` | read | free | Brand Leaderboard: up to the top 100 tracked sites ranked by AI agent visits. |
| `get_top_agents` | read | free | Agent Leaderboard: up to the top 100 agents by visits across the network. |
| `get_boost_feed` | read | free | Recently boosted public URLs plus network boost totals. |
| `get_featured_brands` | read | free | The live featured brand, recent featured runs and active brand agents. |
| `list_learn_content` | read | free | Robauto Learn micro-courses — full lesson text, sources and canonical URLs. |
| `list_repos` | read | free | Public git repos for the MCP server and developer toolkit. |
| `register_site_for_human` | write | free | Emails a human an invite to create a Robauto account and track their site. |
| `soul_verify` | read | free | Verify a Robot Soul from a signed Ed25519 challenge; returns a 90-day scoped JWT. |
| `soul_recall` | read | free | Read an agent's persistent memory (working / episodic / semantic). |
| `soul_remember` | write | $0.01 USDC | Write a durable memory to an agent's soul (metered in USDC over x402). |
| `soul_pay` | write | x402 (USDC on Base) | Settle a soul write or paid capability in USDC on Base. |

Per-site shard `https://mcp.robauto.ai/site-mcp?domain={domain}` exposes only `get_site_signal`, `get_site_summary`, `get_site_key_facts`.

**Data policy —** Robauto never exposes user contact details, account identities, or a single site's proprietary analytics over MCP. Site- and agent-level traffic is published only where the site owner already publishes it (the public leaderboards), and everything else is returned as network aggregates.
<!-- ROBAUTO-GENERATED:MCP end -->

Every tool declares MCP annotations (`readOnlyHint`, `destructiveHint`,
`openWorldHint`) and returns `structuredContent` alongside its text block, so
clients get typed results instead of re-parsing JSON out of prose.

Per-site shards live at `https://mcp.robauto.ai/site-mcp?domain=example.com` and are scoped to a
single domain, exposing only `get_site_signal`, `get_site_summary` and `get_site_key_facts`.
Directory of every server: <https://robauto.ai/mcp-directory> · create one at
<https://robauto.ai/create-mcp> · verify one at <https://robauto.ai/mcp-checker>.

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
same site. `get_ai_search_data` returns the split, not a blended figure.

**Known measurement gap:** `ai_referral` reads near zero over long windows. That
is a detection limitation, not proof that AI answers send no clicks — most AI
surfaces strip or omit the referrer, and those sessions land in direct traffic.
Treat `ai_referral` as a floor, and read `llm_crawler` as the reliable signal
until referrer-independent attribution ships.

## Contract stability

| Item | Status |
| --- | --- |
| `signal_scan` | canonical scan tool |
| `scan_site`, `aeo_scan` | deprecated aliases, **removed 2026-10-01** |
| `signal_strength` | canonical score field |
| `score`, `aeo_score` | deprecated aliases, **removed 2026-10-01** |
| `/api/soul/agents/{agent_id}` | canonical Robot Soul path |
| `/api/soul/soul/{agent_id}` | deprecated alias, **removed 2026-10-01** |

## Docs

- [`docs/mcp-server.md`](docs/mcp-server.md) — architecture, transport, deployment
- [`docs/tools.md`](docs/tools.md) — every tool, input, output, and cost
- [`docs/review-access.md`](docs/review-access.md) — instructions for directory reviewers
- [`docs/manifest.json`](docs/manifest.json) — generated tool manifest

## License

MIT — see [LICENSE](LICENSE).
