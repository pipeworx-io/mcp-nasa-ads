# @pipeworx/nasa-ads

The NASA Astrophysics Data System — essentially the complete bibliography of astronomy and
astrophysics since 1823, with hand-curated citation and reference graphs.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

- `ads_search(query, sort?, limit?, start?, include_abstract?)` — search the literature with the
  full ADS query syntax (`author:"Rubin, V"`, `year:2020-2024`, `bibstem:ApJ`, `abs:"phrase"`,
  `property:refereed`).
- `ads_paper(identifier)` — the full record for one paper by bibcode, DOI or arXiv id. The
  identifier form is detected and searched in the right field, so a DOI never silently free-text
  matches an unrelated paper.
- `ads_citations(bibcode, direction?, sort?, limit?, start?)` — papers citing this one
  (`citations`, default) or its bibliography (`references`).

## Auth

**BYO only.** Pass `_apiKey`. There is **no keyless path**: ADS Bearer-authenticates every
endpoint and an unauthenticated request is a flat `401 Missing "Authorization" in headers`
(verified 2026-09-17). The refusal says "requires an API key" so a caller is not left reading a
401 as our outage.

The token is free and generous: create an account at <https://ui.adsabs.harvard.edu/> then
Settings → API Token (<https://ui.adsabs.harvard.edu/user/settings/token>).

Platform fallback: **`PLATFORM_ADS_TOKEN`**. Until that secret is set, the pack-manifest entry
carries `byoExpected: true`; remove it in the same change that lands the value.

## Data sources

- <https://api.adsabs.harvard.edu/v1/search/query> — every tool here is one call to this endpoint
  with a different `q`. `ads_citations` uses ADS's own `citations(bibcode:...)` and
  `references(bibcode:...)` operators rather than a separate endpoint.

## Traps

- **`fl` is not optional in practice.** Omit it and ADS returns bare ids with no title, author or
  year — a clean 200 that looks like a stripped result set. Every call here sends an explicit field
  list.
- **Unquoted author names are searched as free text.** `author:Rubin` is not an author search;
  `author:"Rubin, V"` is. A free-text match returns plausible, wrong papers with no error, so the
  zero-result note says this explicitly.
- **Year ranges are `year:2020-2024`.** `year>2020` is a syntax error.
- **A bibcode is exactly 19 characters, dot-padded** (`2015ApJ...808...50A`). A copy that lost its
  padding to reflowing matches nothing, which is indistinguishable from "this paper is not indexed".
- **Zero citations is usually real.** For a paper published in the last few months an empty
  `citations` result is ordinary; so is an unknown bibcode. Both return zero rather than an error,
  so the note names both causes.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "nasa-ads": {
      "url": "https://gateway.pipeworx.io/nasa-ads/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/nasa-ads/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ads_search \
  -H 'Content-Type: application/json' \
  -d '{"query":"abs:\"dark energy\" property:refereed","sort":"citation_count desc","limit":20}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ads_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "nasa-ads": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-nasa-ads"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-nasa-ads
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Nasa Ads data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
