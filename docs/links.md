# Deep links and published pages

Two ways to point at something on [roomvana.ai](https://roomvana.ai) without an
account: build a studio link that opens pre-configured, or read a project the user
chose to publish.

## Studio deep links

```
https://roomvana.ai/design?room=bedroom&style=japandi
```

The studio reads four query parameters at mount, validates them against the
catalogue, and applies them once — so a link can never leave the pickers in a
combination no click could produce.

| Parameter | Values | Effect |
|---|---|---|
| `room` | any id from `rooms`, `gardens`, or `exterior` | Selects the space. Passing a garden id also switches the studio to garden mode |
| `style` | any id from `styles` | Pre-selects the design style |
| `surface` | any id from `surfaces` / `exterior_surfaces` | Pre-selects the surface to change (paint a wall, replace a countertop…) |
| `mode` | `surface` | Opens in surface mode instead of full-room redesign |
| `tool` | `floor-plan` | Opens **Floor plan to 3D** — upload a plan (image or PDF), get a furnished 3D view |
| `handoff` | a token from `POST /handoff` | Picks up an image uploaded elsewhere, then deletes the token from the URL |

Only `room` is required to be meaningful; everything else is optional. Unknown ids
are ignored rather than trusted, so an old link degrades to the default studio
instead of erroring.

### Examples

```
https://roomvana.ai/design?room=kitchen&style=modern-farmhouse
https://roomvana.ai/design?room=backyard&style=mediterranean
https://roomvana.ai/design?room=living-room&surface=walls&mode=surface
https://roomvana.ai/design?tool=floor-plan
```

## Published project pages

A user can publish a project from the studio, which mints a slug and serves the
result anonymously:

```bash
curl https://api.roomvana.co/public/shares/<slug>
```

```json
{
  "title": "Living room, japandi",
  "items": [
    {
      "kind": "redesign",
      "room_type": "living-room",
      "name": "Warm oak and linen",
      "before": "<signed image ref>",
      "after": "<signed image ref>"
    }
  ]
}
```

- `before` / `after` are **freshly signed on every read**, so a published page never
  holds an expired reference. They point at `GET /images/{id}`.
- Items whose generation was deleted after publication are skipped silently; the page
  stays servable.
- `GET /public/shares/{slug}/cover` is a **stable** URL that 302s to a fresh reference
  for the first image — built for link previews, which cache `og:image` on their own
  servers and would never come back for a new signature.

The human-facing page for a published project is `/v/<slug>` on roomvana.ai.

## A note on permanence

Deep links and published shares are stable. Hand-off tokens are not — they are
single-use and die after 30 minutes. Use them to move an image between two tools in
the same minute, not to reference it later.
