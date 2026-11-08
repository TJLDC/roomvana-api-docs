# Catalogue — `GET /options`

Every picker on [roomvana.ai](https://roomvana.ai) is built from this one response:
the room list, the style list, which surfaces can be repainted, which paint shades
we ship, and what a generation costs. Fetch it instead of hard-coding ids, and new
styles or rooms appear in your client for free.

## Request

```bash
curl https://api.roomvana.co/options
```

No parameters, no headers, no auth.

## Response

```json
{
  "rooms": ["kitchen", "living-room", "bedroom", "bathroom", "..."],
  "exteriors": ["exterior"],
  "gardens": ["garden", "backyard", "landscape", "patio", "deck", "porch", "balcony"],
  "styles": ["modern", "minimalist", "scandinavian", "japandi", "..."],
  "surfaces": ["walls", "flooring", "wallpaper", "cabinets", "..."],
  "exterior_surfaces": ["siding", "front-door", "garage-door", "shutters", "roof", "deck-boards"],
  "finishes": { "walls": ["warm-white", "greige", "..."], "...": [] },
  "shades": { "walls": { "warm-white": { "brand": "Sherwin-Williams", "code": "SW 7008", "name": "Alabaster" } } },
  "costs": { "basic": 1, "hd": 3, "upscale": 1 },
  "variants_per_generation": 2,
  "daily_free_credits": 0
}
```

### Fields

| Field | Type | Meaning |
|---|---|---|
| `rooms` | `string[]` | Interior spaces. 18 ids, including `closet`, `basement-bar`, `staircase` |
| `exteriors` | `string[]` | Exterior shots. Currently the single id `exterior` |
| `gardens` | `string[]` | Outdoor spaces: `garden`, `backyard`, `landscape`, `patio`, `deck`, `porch`, `balcony` |
| `styles` | `string[]` | 23 design styles, `modern` … `2026-trends` |
| `surfaces` | `string[]` | Interior surfaces that can be changed on their own |
| `exterior_surfaces` | `string[]` | The same idea, outside |
| `finishes` | `object` | Per-surface finish ids (paint colours, wallpaper patterns, cabinet fronts…) |
| `shades` | `object` | For paint finishes, the real product behind the name — brand, code, name |
| `costs` | `object` | Credits per operation: `basic` 1, `hd` 3, `upscale` 1 |
| `variants_per_generation` | `int` | Render count returned per generation (2) |
| `daily_free_credits` | `int` | Credits granted per day without a purchase. Currently `0` |

### Capability flags

The response also carries booleans describing what the studio currently supports, so
a client can hide features it cannot use: `custom_area`, `custom_area_mode`,
`continue_from`, `item_mode`, `custom_prompt`, `floor_plan_mode`, `marks`.

- `item_mode` — add or remove a single piece of furniture rather than redesigning the
  whole room.
- `continue_from` — keep iterating on the previous result instead of starting over.
- `floor_plan_mode` — accept a floor plan (image or PDF) and return a furnished 3D
  bird's-eye view.

## Using the values

Room and style ids are exactly what the studio's deep links expect — see
[Deep links](links.md). If you need a human-readable name for an id, our open-source
clients carry the full label table: [`roomvana` on npm](https://www.npmjs.com/package/roomvana-mcp),
[`roomvana` on PyPI](https://pypi.org/project/roomvana-mcp/), and
[`roomvana` on crates.io](https://crates.io/crates/roomvana).
