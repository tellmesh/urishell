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
