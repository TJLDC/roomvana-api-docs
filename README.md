# Roomvana API documentation

Source of the public API reference published at
**https://roomvana-api.readthedocs.io/** — built by [Read the Docs](https://readthedocs.org)
from this repository.

It covers the part of the Roomvana API that is meant to be called from outside our own
applications: the catalogue, the anonymous image hand-off bridge, the MCP server, deep
links and published project pages. Everything documented there is free, keyless and
rate-limited.

Roomvana turns a photo of a room into a redesigned version of the same room — the same
walls, windows and layout, a different design. The studio is at <https://roomvana.ai>.

## Build locally

```bash
pip install -r requirements.txt
mkdocs serve
```

## Layout

| Path | Purpose |
|---|---|
| `mkdocs.yml` | Site config and navigation |
| `docs/` | The pages |
| `.readthedocs.yaml` | Read the Docs build definition |
| `requirements.txt` | Build dependencies |
