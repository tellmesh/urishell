# urishell

`shell://` URI capability pack — run subprocess commands on the automation host.

Used by:

- `urirdpedge` / `urisys-rdp` (RDP desktop stack)
- `urisys-node` (`pip install urishell`, boot pack `shell`)

```bash
urisys-rdp call 'shell://echo%20hello' --approve
```

## Layout

```text
urishell/
├── urishell/manifest.yaml   # route + handler bindings
├── urishell/handlers.py     # shell_run
└── urishell/routes.py       # register() → manifest
```

Licensed under Apache-2.0.

## Ekosystem TellMesh

Orchestrator: **[urisys](https://github.com/tellmesh/urisys)** · Mapa: **[MESH.md](https://github.com/tellmesh/urisys/blob/main/docs/MESH.md)** · Model: **[ECOSYSTEM.md](https://github.com/tellmesh/urisys/blob/main/docs/ECOSYSTEM.md)**

| Pole | Wartość |
|------|---------|
| **Warstwa** | Capability pack |
| **Scheme** | `shell://` |
| **Zależność** | `uricontrol>=0.1.8` |
| **Edge** | urisys-node, urirdpedge |

Runtime edge: **`uri_control.edge`** w pakiecie **`uricontrol`** (legacy PyPI `uricore` / `urisysedge` usunięty 2026-06).
Resolver intencji: **`uriresolver`** (`uri_resolver`) + transport w **`uritransport`**; policy gate: **`uriguard`** (`uri_guard`).

<!-- end-ecosystem -->
