# 05 — Behavioural consistency

This document explains how to keep **behaviour consistent** between the CLI and the TUI when they perform the same operations. Use the **same core functions** and the **same semantics** (options, defaults) where possible, and **document** where they differ.

---

## Same operations, same core

Where the CLI and TUI do the **same operation** (e.g. “run job X”, “export config”, “list items”), they must call the **same core functions** with the same semantics. That way:

- Behaviour is consistent regardless of interface.
- Bug fixes and improvements in core apply to both.
- You avoid drift (CLI and TUI doing slightly different things).

Do **not** reimplement logic in the CLI and again in the TUI; both should be thin layers over the core.

---

## Same semantics and options

Where an operation supports options (e.g. “run with dry-run”, “limit count”), use the **same option names and defaults** in both interfaces where it makes sense:

- **CLI:** Options come from argv (e.g. `--dry-run`, `--limit 5`).
- **TUI:** The same options can be exposed as toggles, inputs, or dialogs; pass the same values into the core.

If the TUI omits an option (e.g. no “dry-run” in the UI), document that; the core can still accept a default.

---

## Document where they differ

CLI and TUI may **differ by design** in a few places. Document these so users know what to expect:

| Area | CLI | TUI | Note |
|------|-----|-----|------|
| **Output** | Plain text, JSON (`--json`) | Widgets, screens | Document `--json` and format in CLI docs. |
| **Interactive prompts** | Optional (e.g. `typer.Option(prompt=...)`) | Natural in UI | Document when CLI prompts vs TUI-only flows. |
| **Defaults** | Non-interactive defaults (e.g. no confirm) | May add confirmations | Document if TUI has extra confirm steps. |
| **Features** | Scripting-friendly (batch, exit codes) | Rich UI (navigation, live updates) | List TUI-only or CLI-only features. |

List these differences in your user docs or in a “CLI vs TUI” section so that scripting and interactive use are predictable.

---

## Summary

- **Same operations** → same core functions and semantics.
- **Same options and defaults** where possible; document when one interface omits or extends.
- **Document differences** (output format, prompts, confirmations, features) so users know what to expect from each interface.

See also: [02 — Shared core](02-shared-core.md) | [06 — Quick reference](06-quick-reference.md) | [Index](index.md)
