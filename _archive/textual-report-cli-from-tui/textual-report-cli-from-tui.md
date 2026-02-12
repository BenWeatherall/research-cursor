# Archive summary: textual-report-cli-from-tui

## The request

The Textual TUI report had guidance only for **adding a TUI to an existing CLI** (05-cli-integration.md). The feature requested the **reverse**: add a section (or document) covering **adding a CLI to an existing Textual TUI**—when to do it, how to structure entry points and shared logic, and how to keep behaviour consistent. Deliverable: report content only, under `reports/textual-tui-report/`. Validation: human review; no automated tests.

## Planned approach

- **Chosen solution:** New document `06-cli-from-tui.md` (not a subsection in 05); update `index.md` with a row for 06 and optional executive-summary sentence; add one cross-link in 05 to 06.
- **Architecture:** Single new doc; index table and “How to use” updated; 05 gets a “See also” / “Reverse scenario” line. No external libraries; content documents standard Python patterns (argparse, Click, Typer, `__main__.py`, shared core).
- **Task breakdown:** (1) Create 06 with full content outline—when to add CLI, entry-point and structure, shared logic, CLI stack, behavioural consistency, quick reference, summary, links. (2) Update index. (3) Add cross-link in 05.

## What was implemented

- **06-cli-from-tui.md** created with title, intro, “Should you add a CLI?” table, entry-point and structure (directory sketch and Mermaid diagram), shared logic, CLI stack (argparse/Click/Typer; Trogon not applicable), behavioural consistency, quick reference table, summary, and links (Back to index, See also 05). Style aligned with 05; file kept under 500 lines.
- **index.md** updated: new table row for “CLI from TUI” linking to 06; executive summary extended to mention “both adding a TUI to an existing CLI and adding a CLI to an existing TUI”; “How to use this report” updated to reference 06 for adding a CLI to an existing TUI.
- **05-cli-integration.md** updated: “See also: [Adding a CLI to an existing TUI](06-cli-from-tui.md) (reverse scenario).” added before “Back to index”.

All tasks (01–03) completed; report is self-consistent and discoverable from the index and from 05.
