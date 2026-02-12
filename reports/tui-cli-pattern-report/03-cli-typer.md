# 03 — CLI (Typer)

This document covers the **Typer** CLI in `cli.py`: how to define the app and commands, how the entry point invokes it, and how to use type hints and options. Use Typer for the CLI path only; the TUI uses Textual.

---

## Typer app in `cli.py`

Define a single `typer.Typer()` app in `cli.py` and register commands with `@app.command()`. The app is the CLI surface; all subcommands and options live here. Keep **core** logic in `core/` and call into it from command handlers.

**Root help:** Set the app-level `help` so that the root `--help` output includes a note that running the command with **no** arguments launches the TUI. That way users who run `my_tool --help` see how the default (no args) behaves.

```python
"""CLI implemented with Typer."""
from __future__ import annotations

import typer

app = typer.Typer(
    help="My tool: CLI and TUI. Running with no arguments launches the TUI; pass a subcommand or --help for CLI.",
)

@app.command()
def run(job_id: str) -> None:
    """Run a job by ID."""
    from my_tool.core.service import run_job
    result = run_job(job_id, config={})
    print(result)

@app.command()
def list_jobs(limit: int = 10) -> None:
    """List recent jobs."""
    # Call core; print result
    ...
```

---

## Invocation from the entry point

The entry point (`__main__.py`) decides “CLI mode” vs “TUI mode”: no args → TUI; any args → CLI. When there are arguments, the entry point invokes the Typer app so that `sys.argv` (or the appropriate slice) is passed through. Two common patterns:

**Pattern 1: Call `app()` with no args** — Typer reads `sys.argv` by default:

```python
# In __main__.py (CLI branch):
from my_tool.cli import app
app()
```

**Pattern 2: Expose `run_cli()` that calls `app()`** — useful if you need to strip TUI-related args first:

```python
# In cli.py:
def run_cli() -> None:
    app()

# In __main__.py (CLI branch):
from my_tool.cli import run_cli
run_cli()
```

In both cases, Typer parses the remaining argv and runs the matching command.

---

## Commands and options

- **Commands:** Use `@app.command()` for each subcommand; the function **docstring** becomes the short description in `--help`.
- **Options:** Use type hints and defaults; for help and validation use `typer.Option()` and `typer.Argument()` (optionally with `typing.Annotated`).

Example with options:

```python
from typing import Annotated
import typer

@app.command()
def run(
    job_id: Annotated[str, typer.Argument(help="Job ID to run.")],
    dry_run: Annotated[bool, typer.Option(help="Only validate, do not run.")] = False,
) -> None:
    """Run a job by ID."""
    ...
```

---

## Type hints and Annotated

Use **type hints** for all parameters; Typer infers arguments and options from them. For Python 3.9+, prefer `typing.Annotated` with `typer.Argument(...)` or `typer.Option(...)` so help text and defaults are explicit and reusable across commands.

See the project’s **Typer CLI report** (e.g. in `reports/typer-cli-report/`) for: minimal app, multiple commands, dynamic addition of commands with `add_typer()`, and parameter patterns.

---

## Summary

- **Placement:** Typer app and commands in `cli.py`; core in `core/`.
- **Invocation:** Entry point calls `app()` or `run_cli()` when any args are present; Typer parses argv. No args → TUI (see [01 — Entry point and structure](01-entry-point-and-structure.md)).
- **Root help:** App-level `help` on the Typer app should note that running with **no** arguments launches the TUI.
- **Style:** Type hints, docstrings for help, `Annotated` for arguments/options with help and validation.

See also: [01 — Entry point and structure](01-entry-point-and-structure.md) | [02 — Shared core](02-shared-core.md) | [04 — TUI (Textual)](04-tui-textual.md) | [Index](index.md)
