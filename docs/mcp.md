# MCP server

Roomvana ships a [Model Context Protocol](https://modelcontextprotocol.io) server so
an assistant can answer *"make my bedroom japandi"* with a real, pre-configured
studio link instead of a guess. There are two ways to run it — hosted, or local over
stdio — and they expose exactly the same three tools.

## Hosted: `POST /mcp`

```
https://api.roomvana.co/mcp
```

A **stateless Streamable-HTTP** endpoint. One JSON-RPC message in, one JSON response
out. There is no session id, no server-sent stream, and no state between calls.

- `GET /mcp` returns `405` with `Allow: POST` — that is what the Streamable-HTTP spec
  asks of a server that offers no SSE stream.
- A JSON array body is treated as a batch; notifications (messages without an `id`)
  are accepted and answered with `202`.
- Supported protocol versions: `2025-06-18` (default), `2025-03-26`, `2024-11-05`.

```bash
curl -s https://api.roomvana.co/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

Because the server is stateless, `tools/list` and `tools/call` work without a
preceding `initialize`. Clients that do send `initialize` first (most of them) are
answered normally.

## Local: stdio

The same server, as a package you run yourself — useful for editors that only speak
stdio:

```bash
npx roomvana-mcp        # npm — Node 18+
uvx roomvana-mcp        # PyPI — Python 3.10+
```

Point a client at it:

```json
{
  "mcpServers": {
    "roomvana": {
      "command": "npx",
      "args": ["-y", "roomvana-mcp"]
    }
  }
}
```

There is also a [Rust client crate](https://crates.io/crates/roomvana) that reads the
same catalogue, and a [VS Code extension](https://open-vsx.org/extension/roomvana/roomvana-ai-room-design)
that hands an image from the explorer straight to [roomvana.ai](https://roomvana.ai).

## Tools

### `list_design_styles`

No arguments. Returns the design styles we can render — `modern`, `japandi`,
`industrial` and 20 more — each with a short description.

### `list_room_types`

No arguments. Returns the room and outdoor space types we can redesign: interior
rooms, the house exterior, and garden/outdoor spaces.

### `design_a_room`

| Argument | Type | Required | Meaning |
|---|---|---|---|
| `room` | string | yes | A space id, e.g. `bedroom`, `kitchen`, `backyard` |
| `style` | string | no | A style id, e.g. `japandi`, `modern` |
| `notes` | string | no | Free-text wish. Echoed back to the assistant — **not** encoded in the link |

Both ids are validated against the live catalogue before a link is produced; an
unknown id comes back as a tool error listing what is valid. The result is text
containing the studio link:

```
https://roomvana.ai/design?room=bedroom&style=japandi
```

The user opens it, uploads a photo of the room, and generates. Their walls, windows
and layout stay put — only the look changes.

## Design notes

- **Read-only.** All three tools are annotated `readOnlyHint: true`. Nothing here
  creates a project, spends a credit or writes to an account — the heaviest thing it
  does is build a URL.
- **No key.** The hosted endpoint is anonymous and rate-limited like the rest of the
  public surface.
- **Catalogue-driven.** The tool responses are generated from the same `/options`
  payload the website uses, so a new style shows up in your assistant the day it
  ships.
