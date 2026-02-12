# Textual TUI Report — Index

## Executive summary

This report gives teams **standardised guidance** on using the [Textual](https://textual.textualize.io/) library (Python TUI framework) so they can adopt it with consistent patterns, modern Python 3 practices, and clear advice on integrating with existing CLI tooling. It covers app structure, dynamic commands and options, menu systems and traversal, clean exit behaviour, and when and how to combine a TUI with argparse, Click, or Typer—both adding a TUI to an existing CLI and adding a CLI to an existing TUI.

Each document is self-contained and kept under 500 lines; code snippets are valid and aligned with Textual’s current API.

## Document index

| Document | Link | Description |
|----------|------|-------------|
| Index | [index.md](index.md) | This file: entry point and document list. |
| Patterns and structure | [01-patterns-and-structure.md](01-patterns-and-structure.md) | Standardised App/Screen patterns, `compose()`, lifecycle, and Python 3 style. |
| Commands and dynamic options | [02-commands-and-dynamic-options.md](02-commands-and-dynamic-options.md) | System commands, command providers, and dynamic addition of commands/options. |
| Menus and traversal | [03-menus-and-traversal.md](03-menus-and-traversal.md) | Command palette, fuzzy search, traversal, and screen-specific commands. |
| Exit and shutdown | [04-exit-and-shutdown.md](04-exit-and-shutdown.md) | Quit command, key bindings, and programmatic exit. |
| CLI integration | [05-cli-integration.md](05-cli-integration.md) | Whether and how to integrate with argparse/Click/Typer; behavioural consistency when not integrating. |
| CLI from TUI | [06-cli-from-tui.md](06-cli-from-tui.md) | Adding a CLI to an existing TUI; entry points, shared logic, consistency. |

## How to use this report

- **New to Textual:** Start with [01-patterns-and-structure.md](01-patterns-and-structure.md), then [02-commands-and-dynamic-options.md](02-commands-and-dynamic-options.md) and [03-menus-and-traversal.md](03-menus-and-traversal.md) for commands and menus.
- **Exit and CLI:** Use [04-exit-and-shutdown.md](04-exit-and-shutdown.md) for exit behaviour, [05-cli-integration.md](05-cli-integration.md) for adding a TUI to a CLI, and [06-cli-from-tui.md](06-cli-from-tui.md) for adding a CLI to an existing TUI.
- **Reference:** Use the table above to jump to a topic; each document includes code snippets and, where useful, Mermaid diagrams.
