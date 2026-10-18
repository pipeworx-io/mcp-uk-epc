# @pipeworx/uk-epc

UK Energy Performance Certificate (EPC) register MCP — the standard
property energy-rating dataset by postcode. Complements landregistry-uk
(sale prices) with the energy-efficiency figure for the same properties.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1837+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `search_domestic_epc(postcode?, uprn?, address?, council?, constituency?, efficiency_rating?, date_start?, date_end?, page?, page_size?)`
  — domestic (residential) certificates. At least one filter is required.
- `search_non_domestic_epc(...)` — same filters, non-domestic (commercial)
  buildings.
- `get_certificate(certificate_number)` — full record for one certificate
  (domestic EPC, non-domestic EPC, or Display Energy Certificate), headline
  fields promoted plus the complete raw record.

## Auth

**Platform-keyed, and the key does not exist yet.** This register has no
self-service API key — the publisher requires signing in or creating a
GOV.UK One Login account, then copying a bearer token from the account's
"my account" page. That is a government identity-verified login, not
something an agent can register on anyone's behalf. See the header of
`src/index.ts` for what was checked and when.

## Data source

UK Government (Ministry of Housing, Communities and Local Government),
the Energy Performance Certificate register — Energy Performance
Certificates for domestic and non-domestic buildings and Display Energy
Certificates for public buildings, covering England and Wales since 2008.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "uk-epc": {
      "url": "https://gateway.pipeworx.io/uk-epc/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/uk-epc/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1837+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/search_domestic_epc \
  -H 'Content-Type: application/json' \
  -d '{"postcode":"LS1 4AP"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_domestic_epc`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "uk-epc": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-uk-epc"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-uk-epc
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Uk Epc data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
