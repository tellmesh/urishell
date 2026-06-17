# UriPack: urishell

Self-contained Markpact — definitions, full source, run config. Unpack & run: `urisys markpact run urishell/urishell.markpact.md --as service` (writes `.markpact/`).

```yaml markpact:pack
apiVersion: urisys.io/v1
kind: UriPack
metadata:
  id: urishell-pack
  version: 1.0.0
  language: python
description: Subprocess shell commands on the automation host.
schemes:
- shell
capabilities:
- id: shell.run
  uri: shell://{command}
  kind: command
  operation: shell.run
  handler: python://urishell.handlers:shell_run
  side_effects: true
  approval: required
policy:
  default: deny_mutations_without_approval
runtime:
  default_environment: mock
  supports:
  - mock
  - local
  - docker
```

```yaml markpact:run
modes:
- pack
- service
- flow
- interface
- adapter
default: service
scheme: shell
service:
  port: 8790
  wire: POST /uri/call
flow:
  ids:
  - system-apt-update
  - install-chromium
adapter:
  wire: POST /uri/call
  events: GET /events
```

```python markpact:module path=urishell/__init__.py
from __future__ import annotations

from .routes import register

__version__ = "0.1.0"
__all__ = ["register", "__version__"]
```

```python markpact:module path=urishell/handlers.py
"""shell:// — subprocess on automation hosts."""

from __future__ import annotations

import os
import subprocess
from typing import Any


def _allow_real(context: dict[str, Any]) -> bool:
    return bool(context.get("allow_real") or os.environ.get("URISYS_ALLOW_REAL") == "1")


def _detect_display(context: dict[str, Any]) -> str | None:
    if context.get("display"):
        return str(context["display"])
    env = context.get("env_config") or {}
    display = env.get("display") or os.environ.get("DISPLAY")
    return str(display) if display else None


def _mock(command: str, payload: dict[str, Any], context: dict[str, Any]) -> dict[str, Any]:
    args = payload.get("args") or []
    return {
        "driver": "mock",
        "command": command,
        "args": args,
        "display": _detect_display(context),
        "ok": True,
    }


def shell_run(payload: dict[str, Any], context: dict[str, Any]) -> dict[str, Any]:
    params = context.get("params") or {}
    command = str(params.get("command") or payload.get("command") or "")
    args = [str(a) for a in (payload.get("args") or [])]
    if not command:
        raise ValueError("shell command required")

    if context.get("dry_run") or not _allow_real(context):
        return _mock(command, payload, context)

    if command == "apt-get" and os.geteuid() != 0:
        args = [command, *args]
        command = "sudo"

    env = os.environ.copy()
    display = _detect_display(context)
    if display:
        env["DISPLAY"] = display
    xauth = context.get("xauthority")
    if xauth:
        env["XAUTHORITY"] = str(xauth)

    proc = subprocess.run(
        [command, *args],
        capture_output=True,
        text=True,
        env=env,
        timeout=float(payload.get("timeout_s") or 600),
    )
    return {
        "driver": "subprocess",
        "command": command,
        "args": args,
        "exit_code": proc.returncode,
        "stdout": (proc.stdout or "")[-4000:],
        "stderr": (proc.stderr or "")[-2000:],
        "ok": proc.returncode == 0,
    }
```

```python markpact:module path=urishell/routes.py
from __future__ import annotations

from importlib.resources import files

from urisysedge.manifest import register_manifest_file


def register(runtime):
    register_manifest_file(runtime, files(__package__).joinpath("manifest.yaml"))
```

```yaml markpact:flow id=system-apt-update
flow:
  id: system-apt-update
  description: Refresh package index and upgrade installed packages (TUI).

defaults:
  approved: true

do:
  - shell://apt-get:
      args: ["update"]
  - shell://apt-get:
      args: ["upgrade", "-y"]
```

```yaml markpact:flow id=install-chromium
flow:
  id: install-chromium
  description: Install Chromium via apt and verify binary path (TUI).

defaults:
  approved: true

do:
  - shell://apt-get:
      args: ["install", "-y", "chromium-browser"]
  - shell://which:
      args: ["chromium-browser"]
```

```markdown markpact:docs
# urishell

`shell://` URI capability pack — run subprocess commands on the automation host.

Used by:

- `urirdpedge` / `urisys-rdp` (RDP desktop stack)
- `urisys-node` (`pip install urishell`, boot pack `shell`)

~~~bash
urisys-rdp call 'shell://echo%20hello' --approve
~~~

## Layout

~~~text
urishell/
├── urishell/manifest.yaml   # route + handler bindings
├── urishell/handlers.py     # shell_run
└── urishell/routes.py       # register() → manifest
~~~

Licensed under Apache-2.0.
```

