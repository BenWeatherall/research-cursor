# 06 — Quick reference

Scannable reference for the TUI + CLI pattern. For detail, see the linked documents.

---

## Entry point

| Item | Recommendation |
|------|-----------------|
| **Style** | Single script (e.g. `__main__.py`) that branches on argv. |
| **Default (no args)** | No arguments → run Textual `App.run()` (TUI). |
| **CLI path** | Any arguments (subcommand, `--help`, etc.) → run Typer app (e.g. `app()` or `run_cli()`). |
| **Root help** | Typer app-level `help` should note that running with **no** arguments launches the TUI. |
| **Alternative** | Two entry points (e.g. two `console_scripts`); share core; document when to use each. |

See: [01 — Entry point and structure](01-entry-point-and-structure.md)

---

## Package layout

| Path | Role |
|------|------|
| `__main__.py` | Dispatch only; parse argv → CLI or TUI. |
| `cli.py` | Typer app and commands; calls `core/`. |
| `tui/app.py` | Textual App; calls `core/`. |
| `core/` | Config, validation, business logic; UI-agnostic. |

See: [01 — Entry point and structure](01-entry-point-and-structure.md)

---

## CLI stack

| Item | Choice |
|------|--------|
| **Library** | Typer (type-hint CLI; built on Click). |
| **Placement** | `cli.py` only; entry point invokes it in CLI mode. |
| **Style** | Type hints, docstrings for help, `Annotated` for args/options. |

See: [03 — CLI (Typer)](03-cli-typer.md)

---

## TUI stack

| Item | Choice |
|------|--------|
| **Library** | Textual. |
| **Placement** | `tui/app.py` (or `tui/` module); entry point calls `App().run()` in TUI mode. |
| **Style** | `compose()`, type hints; call core from handlers; show errors in UI. |

See: [04 — TUI (Textual)](04-tui-textual.md)

---

## Behavioural consistency

| Rule | Action |
|------|--------|
| **Same operation** | Use same core function and semantics in CLI and TUI. |
| **Same options** | Same option names and defaults where possible. |
| **Differences** | Document (e.g. `--json`, interactive vs non-interactive, TUI-only features). |

See: [02 — Shared core](02-shared-core.md) | [05 — Behavioural consistency](05-behavioural-consistency.md)

---

## Checklist

- [ ] Single entry point (or two documented entry points) that dispatch to CLI or TUI.
- [ ] No args → TUI (default); any args → CLI.
- [ ] Root CLI help (Typer app `help=`) notes that running with no arguments launches the TUI.
- [ ] `core/` holds config, validation, and business logic; no UI branching.
- [ ] CLI in `cli.py` (Typer); TUI in `tui/app.py` (Textual).
- [ ] CLI handles exceptions and `sys.exit()`; TUI shows errors in the UI.
- [ ] Same operations use same core; document where CLI and TUI differ.

[Index](index.md)
