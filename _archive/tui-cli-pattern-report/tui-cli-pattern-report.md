# Archive summary: tui-cli-pattern-report

## The request

Teams needed a single, practical report that establishes a reusable pattern for adding both a TUI (Textual) and a CLI (Typer) to a library project. Existing reports covered Textual and Typer separately; the deliverable was to unify these into one consistent pattern: single entry point dispatching to CLI or TUI, shared core logic, with code examples, mermaid diagrams, and advice aligned with best practices and modern Python 3. Testing was validation-only (structure, example validity, diagram render, consistency); no automated tests.

## Planned approach

Research selected a **multi-document report** under `reports/tui-cli-pattern-report/` with a **single entry point** (e.g. `__main__.py`) dispatching to Typer CLI or Textual TUI based on argv, and a **shared core** layer. Architecture: index plus six documents (01 entry-point and structure, 02 shared core, 03 CLI Typer, 04 TUI Textual, 05 behavioural consistency, 06 quick reference). Eight tasks were defined: create report directory, write documents 01–06, write index. Task plans specified overview, files, test strategy, implementation order, and validation for each task.

## What was implemented

All eight tasks were completed. The report was created at `reports/tui-cli-pattern-report/` with: **index.md** (executive summary, document index, how to use); **01-entry-point-and-structure.md** (when to add TUI/CLI, single entry point, package layout, mermaid dispatch flow, code example); **02-shared-core.md** (core role, no UI branching, CLI exit vs TUI errors, core and CLI code example); **03-cli-typer.md** (Typer in cli.py, invocation from entry point, commands/options, type hints); **04-tui-textual.md** (Textual App in tui/, compose, invocation, calling core from TUI); **05-behavioural-consistency.md** (same operations/same core, document differences); **06-quick-reference.md** (tables and checklist). Code examples are type-hinted and align with the described layout; the dispatch flowchart is in Mermaid. The report references the existing Textual and Typer reports for deeper patterns. No application code or tests were added; the project test suite (empty) was run and passed before commit.
