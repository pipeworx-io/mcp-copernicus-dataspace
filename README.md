# @pipeworx/copernicus-dataspace

European Sentinel and Copernicus Contributing Mission archive from the EU's
Copernicus Data Space Ecosystem — search Sentinel-1/2/3/5P, Copernicus DEM,
land-cover and burnt-area products by area and date and get item ids,
footprints, dates and product asset paths.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1875+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `copernicus_dataspace_collections(contains?, prefix?, limit?)` — the product
  collections, with licence, extent and date range. Call this first: search
  refuses a query that names no collection.
- `copernicus_dataspace_collection(collection_id)` — one collection in full.
- `copernicus_dataspace_search(collections, bbox?, datetime?, ids?, limit?, max_assets?)`
  — which products cover an area on given dates.

## Auth

Catalogue search is keyless. **Downloading a product is not:** asset hrefs are
`s3://eodata/...` object-store paths, and fetching the bytes needs a free
Copernicus account plus S3 or OData credentials (register at
<https://dataspace.copernicus.eu>). The ids, dates, footprints and asset
inventory returned here are complete and need no account.

## Related packs

The `sentinel-hub` pack is the **commercial** Sentinel Hub processing API (OAuth
client credentials; statistical/NDVI endpoints returning computed values). This
pack is the free EU catalogue underneath it — it says which scenes exist;
Sentinel Hub renders and analyses them. Neither substitutes for the other.

## Data sources

- <https://catalogue.dataspace.copernicus.eu/stac> — STAC API 1.0. It redirects
  its own `links` to `https://stac.dataspace.copernicus.eu/v1`; both answer.

Shared client: `shared/src/stac.ts`, also used by `planetary-computer`,
`earth-search` and `meteoswiss`.

## Traps

- `POST /search` with no `collections` answers **HTTP 400
  `CollectionInQuerryIsMissingError`** (the upstream's own spelling).
- **`GET /collections` pages ten at a time.** A single un-paged read looks like a
  ten-dataset catalogue — a confident wrong answer rather than an error. The
  shared client follows `rel="next"`.
- Asset hrefs are `s3://`, not HTTPS. An agent that treats one as a download URL
  gets nothing and reads it as a broken link.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "copernicus-dataspace": {
      "url": "https://gateway.pipeworx.io/copernicus-dataspace/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/copernicus-dataspace/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1875+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/copernicus_dataspace_collections \
  -H 'Content-Type: application/json' \
  -d '{"contains":"sentinel-2","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/copernicus_dataspace_collections`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "copernicus-dataspace": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-copernicus-dataspace"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-copernicus-dataspace
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Copernicus Dataspace data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
