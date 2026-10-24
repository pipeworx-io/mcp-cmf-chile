# @pipeworx/cmf-chile

Chile's financial regulator, the Comisión para el Mercado Financiero (CMF): sanction resolutions, regulations (NCG, circulares, oficios circulares) and the registers of supervised entities and Fintec financial service providers.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1878+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `cmf_sanctions(query?, year?, limit?)` — the CMF's register of sanction resolutions (about 1,500, 2002 to today): number, date, subject and a link to the resolution document. Answers "has the CMF sanctioned X?". A miss returns `found:false` with the scope searched.
- `cmf_regulation(type?, number?, query?, year?, limit?)` — resolve a norm (`type=NCG number=514`) to its title, issue date, PDF link, approving resolution and the norms that modify or repeal it, or list norms by keyword and year.
- `cmf_registry(rut?, name?)` — look up a legal entity in the supervised-entity register and in the Fintec register of financial service providers (which services it is authorised for, and whether the registration is current). RUT may be written `77.532.023-0` or `77532023`.

## Auth

Keyless. Every call is a live per-request read of the CMF's public website; nothing is stored or mirrored.

## Data sources

- Comisión para el Mercado Financiero (CMF), Chile — register of sanction resolutions.
- Comisión para el Mercado Financiero (CMF), Chile — legislation and regulations (NCG, circulares, oficios circulares).
- Comisión para el Mercado Financiero (CMF), Chile — register of supervised entities, and the register of financial service providers under the Fintec law.

Every response carries `source`, `data_as_of` and `scope_searched`.

## Notes

- **Natural persons are never served.** Sanction resolutions and some registers name individuals. This pack serves legal entities only: a sanction row whose subject is not recognisably a company, fund or institution keeps its number, date and action but the name and document link are withheld (`subject_withheld: true`), and `withheld_natural_person_rows` counts them. Registrants identified by a personal RUT are dropped the same way. The test is conservative, so a few real entities with unusual names may be withheld.
- `found:false` means "not in these registers", not "does not exist". A sanction announced only in a press release is not in the resolutions register and is not searched.
- The sanctions register is one long page; the filter runs on our side after fetching it, so each call costs one fetch regardless of the query.
- Spanish input works best (`"finanzas abiertas"`, `"Cooperativa"`); matching is accent- and case-insensitive.
- `cmf_registry` reads two registers independently. If the CMF refuses one from the region the call runs in, the other still answers and the response carries `partial: true` and `registers_unavailable`; `found:false` on a partial answer is not evidence the entity is unregistered. If both are refused the call fails and says so.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "cmf-chile": {
      "url": "https://gateway.pipeworx.io/cmf-chile/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/cmf-chile/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1878+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/cmf_sanctions \
  -H 'Content-Type: application/json' \
  -d '{"query":"Itaú","year":2026}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/cmf_sanctions`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "cmf-chile": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-cmf-chile"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-cmf-chile
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Cmf Chile data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
