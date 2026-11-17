# @pipeworx/in-sebi-orders

Has the Securities and Exchange Board of India issued an enforcement order naming this company, broker or fund? The newest adjudication, Chairperson/Members and settlement orders that name a firm.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2014+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `in_sebi_orders_list(category?, name?)` — the board's 25 newest orders in one category (`adjudication` default, `chairperson_members`, `settlement_board`): date, title and link. `name` keeps rows whose title contains it. Returns `result` (`rows`, `no_title_match_on_first_page`, `no_firm_order_on_first_page`), `orders_scanned`, `person_or_unclassified_dropped`, `unparsed_rows`, `oldest_order_date_on_page`, `total_orders_in_category`, `coverage` and `data_as_of`. `link_kind` says whether the link is an order page or the order document itself.
- `in_sebi_orders_read(link)` — an order's own page: title, date, category and the link to the order document that holds the findings, the penalty and the noticees. The page carries no order text.

## Auth

Keyless.

## Data sources

- Securities and Exchange Board of India, enforcement orders, read from the publisher on request. `data_as_of` is the day of the read; order dates are the board's own.

## Behaviour worth knowing

- Firms only. The board's lists mix companies, intermediaries and funds with individuals (research analysts, individual brokers). A row whose title carries no company, intermediary or fund marker is dropped and counted in `person_or_unclassified_dropped`.
- A firm named in a title is the subject of the matter, not necessarily every noticee: the order document lists them.
- Only the first page of each category is read, because the board's paging and search run through page scripts that are not used. An older order is not found here, and no row is not a clearance.
- A page that cannot be read or has changed shape is an error, never "no order found".

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "in-sebi-orders": {
      "url": "https://gateway.pipeworx.io/in-sebi-orders/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/in-sebi-orders/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2014+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/in_sebi_orders_list \
  -H 'Content-Type: application/json' \
  -d '{"category":"adjudication"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/in_sebi_orders_list`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-in-sebi-orders` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/in-sebi-orders/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "in-sebi-orders": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-in-sebi-orders"]
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
ask_pipeworx({ question: "your question about In Sebi Orders data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
