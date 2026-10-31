# @pipeworx/cmf-chile

Chile's financial regulator, the Comisión para el Mercado Financiero (CMF): sanction resolutions, regulations (NCG, circulares, oficios circulares) and the registers of supervised entities and Fintec financial service providers.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1863+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

- **Individuals named in sanction resolutions are served (Bruce's case ruling, 2026-10-25, scoped to `cmf_sanctions`), on the same terms as `sos-entities`.** Every row carries its source record (`source_record`: resolution number, date, CMF document link). No address is ever served for an individual: rows are an allowlist with no address field, and a natural person's RUT is stripped from the subject text (company RUTs stay). Anyone named can ask for removal at support@pipeworx.io; a removed name is replaced in the subject by `[nombre retirado a solicitud]`, the document link (which names the person) is dropped, other parties on the same resolution are kept, the name stops matching searches, and `withheld_by_removal_request` counts the rows.
- **`subject_kind`** is `person`, `organisation`, `mixed` or `unclassified`, assigned only from explicit cues in the subject text: an honorific (don, doña, señor/a, sr/a) marks a person, a corporate-form or institutional word (S.A., Limitada, Banco, Seguros...) marks an organisation, both marks `mixed`. A bare personal name with neither cue (for example "ANDRÓNICO LUKSIC CRAIG") is `unclassified`; it is not guessed from the shape of the name, and `subject_kind_basis` states the reason. `capacity_stated` (asesor previsional, agente de ventas...) appears only when the subject says it in words. Earlier versions withheld any row without a corporate-form word, which hid real companies and every person; that filter is gone.
- `cmf_registry` is unchanged: it still returns legal entities only, because the ruling names the sanctions register.
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

Both URLs reach the same gateway and the same 1863+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/cmf_sanctions \
  -H 'Content-Type: application/json' \
  -d '{"query":"Itaú","year":2026}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/cmf_sanctions`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-cmf-chile` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/cmf-chile/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
