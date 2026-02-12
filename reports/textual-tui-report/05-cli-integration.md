# 05 — CLI integration

This document gives **established advice** on integrating a Textual TUI with existing command-line tooling (argparse, Click, Typer): whether you should integrate, how to do it when recommended, and how to keep behaviour consistent when you keep CLI and TUI separate.

---

## Should you integrate?

| Situation | Recommendation |
|-----------|-----------------|
| **New TUI-only tool** | No CLI integration required; optional CLI args (e.g. `--headless`) only if needed. |
| **Existing CLI and you want a TUI mode** | Integrate so one codebase serves both: CLI for scripts/CI, TUI for interactive use. |
| **TUI-first with optional CLI** | Same as above: single entry point that branches on flags (e.g. `--tui` vs default CLI). |
| **Different behaviour by design** | Keep separate entry points and share **logic** (parsers, services); document behavioural differences. |

**Integrate** when you want one set of options, one codebase, and consistent behaviour between CLI and TUI where possible. **Don’t integrate** when the TUI is a separate product or when CLI and TUI are meant to behave differently; then share only core logic and document the split.

---

## How to integrate

### Option 1: Trogon (Click → TUI)

**[Trogon](https://github.com/Textualize/trogon)** (from Textualize) turns a **Click** CLI into a TUI: it introspects your Click app and builds a Textual UI for it. Best when your CLI is already in Click.

- **Pros**: Little extra code; consistent options between CLI and TUI; official Textual ecosystem.
- **Cons**: Click-only; you don’t hand-design every TUI screen.
- **Use when**: You have or can use Click and want a generated TUI.

### Option 2: Typer (Click under the hood)

**Typer** builds on Click, so Trogon can drive a Typer app as well. If you prefer Typer’s type-hint style, use Typer for the CLI and run it under Trogon for the TUI.

- **Pros**: Type-hint CLI; same Trogon integration as Click.
- **Cons**: Same as Trogon (generated TUI, not custom screens).
- **Use when**: You want Typer + a generated TUI.

### Option 3: argparse + argparse-tui

**[argparse-tui](https://github.com/fresh2dev/argparse-tui)** (community) provides a TUI layer for **argparse**-based CLIs. Use when your CLI is built with argparse and you don’t want to switch to Click.

- **Pros**: Fits existing argparse apps; no Click dependency.
- **Cons**: Community-maintained; may lag argparse or Textual changes.
- **Use when**: CLI is argparse and you want a generated TUI.

### Option 4: Custom entry point (shared logic)

Keep **two entry points**: one that runs the CLI (argparse/Click/Typer), one that runs the Textual app. Share **logic** (parsing, config, services) in a common module; only the “main” and I/O differ.

```text
my_tool/
  __main__.py     # Dispatches to cli.run() or tui.run() based on argv
  cli.py          # argparse/click/typer
  tui/
    app.py        # Textual App
  core/
    config.py     # Shared
    service.py    # Shared
```

- **Pros**: Full control over TUI and CLI; clear separation; same behaviour where you share code.
- **Cons**: You maintain two UIs (CLI and TUI) and the dispatch logic.
- **Use when**: You need a custom TUI and/or different UX between CLI and TUI.

---

## Behavioural consistency when not integrating

If you **don’t** use Trogon/argparse-tui (i.e. you don’t auto-generate the TUI from the CLI):

- **Share core logic**: Parsing, validation, config, and business logic in a single place; CLI and TUI both call into it.
- **Document differences**: List where the TUI does more/less than the CLI (e.g. interactive only, different defaults).
- **Same semantics**: Where they do the same thing (e.g. “run job X”), use the same functions and options so behaviour stays consistent.
- **Entry points**: One script/module for CLI, one for TUI; optional wrapper that chooses based on `--tui` or env.

---

## Quick reference

| CLI stack | TUI integration option | Behavioural consistency |
|-----------|------------------------|--------------------------|
| **Click** | Trogon (recommended) | Same options and behaviour by default. |
| **Typer** | Trogon (Typer = Click) | Same as Click. |
| **argparse** | argparse-tui or custom | argparse-tui: generated; custom: share logic and document. |
| **Custom / none** | Custom Textual app only | N/A; keep TUI and any CLI in sync via shared core. |

---

## Summary

- **Integrate** when you want one codebase and one set of options for CLI and TUI: use **Trogon** for Click/Typer, **argparse-tui** for argparse, or a **custom** dual entry point with shared logic.
- **Don’t integrate** when TUI and CLI are separate products or intentionally different; then **share logic** and **document** where behaviour differs.
- In all cases: **shared core logic** and **clear documentation** keep CLI and TUI behaviour consistent.

See also: [Adding a CLI to an existing TUI](06-cli-from-tui.md) (reverse scenario).

---

[Back to index](index.md)
