# TUI + CLI pattern report — Index

## Executive summary

This report establishes a **single, practical pattern** for adding both a **TUI** (Textual) and a **CLI** (Typer) to a library project. Use one **entry point** (e.g. `__main__.py`) that parses argv: **no arguments** runs the TUI (default); **any arguments** run the Typer CLI. The root CLI help should note that running with no arguments launches the TUI. Keep **shared logic** in a **core** layer so behaviour stays consistent. The pattern is reusable, uses modern Python 3 and type hints, and aligns with Textual and Typer best practices.

---

## Document index

| Document | Link | Description |
|----------|------|-------------|
| Entry point and structure | [01-entry-point-and-structure.md](01-entry-point-and-structure.md) | When to add TUI/CLI; single entry point; TUI default when no args; package layout; dispatch flow; code and diagrams. |
| Shared core | [02-shared-core.md](02-shared-core.md) | Core layer (config, validation, business logic); UI-agnostic; CLI exit and TUI errors; code example. |
| CLI (Typer) | [03-cli-typer.md](03-cli-typer.md) | Typer app in `cli.py`; root help notes no-args → TUI; commands and options; type hints and Annotated. |
| TUI (Textual) | [04-tui-textual.md](04-tui-textual.md) | Textual App in `tui/`; compose and screens; invocation from entry point; calling core from the TUI. |
| Behavioural consistency | [05-behavioural-consistency.md](05-behavioural-consistency.md) | Same operations use same core; same semantics; document where CLI and TUI differ. |
| Quick reference | [06-quick-reference.md](06-quick-reference.md) | Tables and checklist for entry point, layout, CLI stack, TUI stack, consistency. |

---

## How to use this report

- **New to the pattern:** Start with [01 — Entry point and structure](01-entry-point-and-structure.md), then [02 — Shared core](02-shared-core.md), then [03 — CLI (Typer)](03-cli-typer.md) and [04 — TUI (Textual)](04-tui-textual.md) for implementation details.
- **Consistency and reference:** Use [05 — Behavioural consistency](05-behavioural-consistency.md) when aligning CLI and TUI behaviour, and [06 — Quick reference](06-quick-reference.md) for a fast lookup.
- **Existing reports:** For deeper Textual or Typer patterns (commands, screens, parameters), see the project’s Textual TUI report and Typer CLI report where available.
