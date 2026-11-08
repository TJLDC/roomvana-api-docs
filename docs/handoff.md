# Image hand-off — `POST /handoff`

The hand-off bridge exists for one situation: **the photo is already inside some
other tool, and there is no Roomvana account there to attach it to.** A Figma frame,
a Canva export, a Photoshop canvas, a floor-plan PDF. You post the image, you get a
short-lived token, you open [roomvana.ai](https://roomvana.ai) with that token, and
the studio picks the image up as if it had been uploaded there.

No account. No API key. No credit is spent — this endpoint only moves bytes.

## 1. Upload

```bash
curl -X POST https://api.roomvana.co/handoff \
  -F "file=@/path/to/room.jpg"
```

### Request

- `multipart/form-data`, a single field named **`file`**
- Accepted: any `image/*`, or `application/pdf` (the first page is rasterised)
- Maximum **20 MB** on the wire — the image is then downscaled so its long edge is
  at most 1536 px, and re-encoded as JPEG. Only real, decodable images survive this
  step; anything else is rejected instead of stored.
- Rate limit: **30 uploads per hour per IP**
- CORS: the response carries `Access-Control-Allow-Origin: *`, because editor plugins
  run their UI in an iframe with an opaque origin (`null` in Figma) and would
  otherwise be unable to read the reply. The endpoint is anonymous and carries no
  credentials, so this costs nothing.

### Response

```json
{ "token": "3f9c1a...", "expires_in": 1800 }
```

A PDF adds `"pages": N` — the number of pages in the document, of which only the
first becomes an image.

### Errors

| Status | When |
|---|---|
| `400` | Not an image and not a PDF; empty body; an image or PDF we cannot decode |
| `413` | Larger than 20 MB |
| `429` | More than 30 hand-offs from this IP in the last hour |

## 2. Open the studio with the token

```
https://roomvana.ai/design?handoff=<token>
```

The studio reads the token **once** via `GET /handoff/{token}`, then drops it. The
user sees the image in the upload slot, crops it if they want, picks a room and a
style, and generates.

## 3. Reading it back

```bash
curl -OJ https://api.roomvana.co/handoff/<token>
```

Returns the image bytes with `Cache-Control: no-store`. **The second read is a 404** —
the entry is deleted on first read, and in any case it expires 30 minutes after
creation. Treat the token as a one-shot capability, not an id.

!!! warning "Do not build anything durable on a hand-off token"
    It is not a share link and not a file id. It is designed to live for seconds:
    created by a plugin, consumed by the studio, forgotten. If you need a link that
    survives, publish the project from the studio instead — that is what
    `GET /public/shares/{slug}` serves.

## Why it is shaped this way

An open, anonymous image sink needs fences. Ours: a per-IP rate limit, a hard size
cap, image-only decoding and re-encoding (so only downscaled real images are ever
stored), read-once semantics, a 30-minute TTL, and storage that is in-memory and
bounded. Generation itself never happens here — it stays on the website, behind the
user's own account and credits.
