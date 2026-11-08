# Roomvana API

Roomvana redesigns a room from a photo: **the same room, the same windows, the same
layout — a different design.** It works on interiors, exteriors, gardens and floor
plans, in 23 styles. The studio itself is at [roomvana.ai](https://roomvana.ai).

This site documents the part of our API that is meant to be called from outside our
own applications. Everything here is anonymous, rate-limited and free: no API key,
and nothing documented on this site spends a credit.

## Base URL

```
https://api.roomvana.co
```

HTTPS only. Responses are JSON unless stated otherwise.

## The public surface

| Endpoint | What it does |
|---|---|
| [`GET /options`](catalog.md) | The catalogue: room ids, style ids, surfaces, credit costs |
| [`POST /handoff`](handoff.md) | Upload one image (or PDF), get a one-time token |
| [`GET /handoff/{token}`](handoff.md) | Read that image back once, then it is gone |
| [`POST /mcp`](mcp.md) | A stateless Model Context Protocol server |
| [`GET /public/shares/{slug}`](links.md#published-project-pages) | A project the user chose to publish |
| `GET /health` | Liveness probe |

!!! note "What is deliberately not here"
    Accounts, generation, billing and admin routes exist, but they are used only by
    the Roomvana web app and the iOS app. They are not an integration surface, and
    they are not documented on purpose.

## Three ways in

1. **Your image lives in another tool** — a Figma frame, a Canva export, the first
   page of a floor-plan PDF. Post it to `POST /handoff`, then open the studio with
   the token. → [Image hand-off bridge](handoff.md)
2. **You are building an agent or an editor plugin** — speak MCP to `POST /mcp` over
   Streamable HTTP, or run the same server locally with `npx roomvana-mcp` or
   `uvx roomvana-mcp`. → [MCP server](mcp.md)
3. **You just want a pre-filled link** — the studio reads room, style and surface
   from the query string. → [Deep links](links.md)

## Conventions

- **Ids, not labels.** Every catalogue value is a lowercase hyphenated id
  (`living-room`, `mid-century-modern`). Display names live in the clients and SDKs,
  not in the API.
- **No authentication.** There is no token to obtain and no key to rotate.
- **Errors are JSON** with FastAPI's usual shape: `{"detail": "..."}` plus a
  meaningful status code (`400`, `404`, `413`, `429`).
- **Be a good citizen.** The hand-off endpoint accepts 30 uploads per hour per IP.
  The MCP endpoint is stateless — one request in, one response out.
