# 02 — Shared core

This document describes the **shared core layer**: config, validation, and business logic that both the CLI and the TUI use. The core is **UI-agnostic**; it does not branch on “am I CLI or TUI?” and does not import from `cli` or `tui`. The **CLI** is responsible for catching exceptions, printing messages, and calling `sys.exit(code)`; the **TUI** may show errors in the UI.

---

## Role of the core

Put the following in a **library layer** (e.g. `core/`):

- **Config**: Loading and validating configuration (files, env, defaults).
- **Validation**: Input validation and business rules; raise on error.
- **Business logic**: Operations that both CLI and TUI need (e.g. “run job”, “export data”).

Both the CLI and the TUI **call into** this layer; neither duplicates its logic. The core **raises** on errors; it does not print to stdout/stderr or call `sys.exit()`.

---

## No UI branching in core

Do **not** branch on “am I CLI or TUI?” inside the core. Keep the core independent of the interface:

- **Good:** Core exposes functions that take data and return results or raise; CLI and TUI each decide how to present results or handle errors.
- **Bad:** Core checks a flag or global to decide whether to print or show a dialog.

This keeps the core testable and reusable from other entry points (e.g. tests, another script).

---

## CLI vs TUI responsibilities

| Responsibility | CLI | TUI |
|----------------|-----|-----|
| **Call core** | Yes; pass parsed args/config. | Yes; pass state from UI. |
| **On success** | Print result (e.g. to stdout); exit 0. | Update UI; show result. |
| **On error** | Catch exception; print message; `sys.exit(non_zero)`. | Catch exception; show error in UI (e.g. dialog, banner). |
| **Output format** | Plain text, JSON (e.g. `--json`), etc. | Widgets, screens. |

The core **raises**; the **CLI** handles exit and output; the **TUI** handles presentation.

---

## Code example: core and CLI caller

**`core/service.py`** — core logic; raises on error:

```python
"""Shared business logic; UI-agnostic."""
from __future__ import annotations


class CoreError(Exception):
    """Raised when a core operation fails."""
    pass


def run_job(job_id: str, config: dict[str, str]) -> str:
    """Run a job; returns result message. Raises CoreError on failure."""
    if not job_id or not job_id.strip():
        raise CoreError("job_id must be non-empty")
    # ... perform work using config ...
    return f"Job {job_id} completed"
```

**`cli.py`** — CLI calls core and handles exit:

```python
"""CLI entry; catches core errors and exits."""
from __future__ import annotations

import sys
import typer

from my_tool.core.service import run_job, CoreError

app = typer.Typer()


@app.command()
def run(job_id: str) -> None:
    """Run a job by ID."""
    try:
        result = run_job(job_id, config={})
        print(result)
    except CoreError as e:
        print(f"Error: {e}", file=sys.stderr)
        raise SystemExit(1)


def run_cli() -> None:
    """Entry point for CLI path from __main__."""
    app()
```

The TUI would also call `run_job()` and catch `CoreError` to show an error message in the UI instead of exiting.

---

## Summary

- **Core:** Config, validation, business logic; raises on error; no UI branching.
- **CLI:** Calls core; catches exceptions; prints and `sys.exit(code)`.
- **TUI:** Calls core; catches exceptions; shows errors in the UI.

See also: [01 — Entry point and structure](01-entry-point-and-structure.md) | [03 — CLI (Typer)](03-cli-typer.md) | [04 — TUI (Textual)](04-tui-textual.md) | [Index](index.md)
