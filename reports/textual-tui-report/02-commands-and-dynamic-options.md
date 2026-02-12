# 02 — Commands and dynamic options

This document covers **system commands** and **command providers** in Textual: how to add commands to the command palette and how to supply **dynamic** commands (e.g. that depend on context or I/O). All snippets use the current Textual API.

References: [Command palette guide](https://textual.textualize.io/guide/command_palette/), [API: command](https://textual.textualize.io/api/command/), [API: system_commands](https://textual.textualize.io/api/system_commands_source).

---

## Command palette

The **command palette** is a built-in screen opened (by default) with **Ctrl+P**. It shows a search input and a list of commands; the user types to filter (fuzzy search), moves with Up/Down, and runs a command with Enter. Commands come from:

1. **System commands** — from `App.get_system_commands(screen)` (and base class).
2. **Command providers** — from `App.COMMANDS` (e.g. custom `Provider` classes).

---

## System commands

**System commands** are the simplest way to add commands: implement `get_system_commands()` on your **App** and yield `SystemCommand` instances. You receive the **screen** that was active when the palette was opened, so you can tailor commands to the current screen.

Signature:

```python
def get_system_commands(self, screen: Screen) -> Iterable[SystemCommand]:
    ...
```

Each **SystemCommand** has:

- **title**: Shown in the palette (e.g. `"Bell"`).
- **help**: Short description (e.g. `"Ring the bell"`).
- **callback**: Callable run when the user selects the command (e.g. `self.bell`).
- **discover** (optional): If `True` (default), the command is shown even when the search input is empty; if `False`, it appears only when there is search text.

Example: add a “Bell” command while keeping the default commands (Quit, Theme, etc.):

```python
from __future__ import annotations

from typing import Iterable

from textual.app import App, SystemCommand
from textual.screen import Screen


class BellCommandApp(App[None]):
    def get_system_commands(self, screen: Screen) -> Iterable[SystemCommand]:
        yield from super().get_system_commands(screen)
        yield SystemCommand("Bell", "Ring the bell", self.bell)
```

- Always `yield from super().get_system_commands(screen)` so default commands (Quit, Theme, Screenshot, etc.) remain.
- Callbacks can be methods (`self.bell`) or lambdas; they are invoked with no arguments when the user selects the command.

**Discover flag** — hide a command until the user types:

```python
yield SystemCommand("Niche", "Only when searching", self.niche_action, discover=False)
```

---

## Command providers (dynamic commands)

For **dynamic** commands (e.g. depending on files, network, or the current screen), use a **command provider**: a class that implements `textual.command.Provider` and is registered on the App via the **COMMANDS** class variable.

Provider methods (all async):

- **`search(self, query: str) -> Hits`**: Called on each keypress with the current search string. Yield `Hit(score, highlighted_text, callback, help="...")` for each matching command. Use `self.matcher(query)` to get a fuzzy matcher; `matcher.match(text)` returns a score (0 = no match).
- **`discover(self) -> DiscoveryHits`** (optional): Called when the palette is opened with empty input. Yield `DiscoveryHit(...)` for commands that should be visible by default (keep this fast).
- **`startup(self)`** (optional): Called once when the palette is opened. Use for loading data (e.g. run a worker, await result) so `search()` can use it.
- **`shutdown(self)`** (optional): Called when the palette closes. Use to clean up resources created in `startup()`.

Example: provider that offers “Open &lt;file&gt;” for each `.py` file in the current directory. Data is loaded in `startup()`; `search()` uses the matcher and yields `Hit` with a callback.

```python
from __future__ import annotations

from functools import partial
from pathlib import Path

from textual.app import App, ComposeResult
from textual.command import Hit, Hits, Provider
from textual.containers import VerticalScroll
from textual.widgets import Static


class PythonFileCommands(Provider):
    async def startup(self) -> None:
        worker = self.app.run_worker(self._read_files, thread=True)
        self._paths: list[Path] = await worker.wait()

    def _read_files(self) -> list[Path]:
        return list(Path(".").glob("*.py"))

    async def search(self, query: str) -> Hits:
        matcher = self.matcher(query)
        for path in self._paths:
            command = f"open {path}"
            score = matcher.match(command)
            if score > 0:
                yield Hit(
                    score,
                    matcher.highlight(command),
                    partial(self.app.open_file, path),
                    help="Open this file",
                )


class ViewerApp(App[None]):
    COMMANDS = App.COMMANDS | {PythonFileCommands}

    def compose(self) -> ComposeResult:
        with VerticalScroll():
            yield Static(id="content")

    def open_file(self, path: Path) -> None:
        self.query_one("#content", Static).update(str(path))
```

- **COMMANDS**: Merge with `App.COMMANDS` so default providers stay: `COMMANDS = App.COMMANDS | {MyProvider}`.
- **Hit**: `(score, display_text, callback, help=...)`. `display_text` can be the result of `matcher.highlight(text)` for match highlighting.
- **Callback**: Can be a method on the app (e.g. `partial(self.app.open_file, path)`).

**Screen-specific commands**: Add a `COMMANDS` class variable on a **Screen** subclass; those providers are used only when that screen is active.

---

## Dynamic addition summary

| Mechanism | Use case |
|-----------|----------|
| `get_system_commands()` | Fixed set of app-wide commands; simple callbacks. |
| `Provider` in `App.COMMANDS` | Dynamic or many commands; async search/startup; screen-aware. |
| `COMMANDS` on a Screen | Commands only when that screen is active. |

- Prefer **system commands** for a small, fixed set.
- Prefer **providers** when commands depend on I/O, context, or large/variable sets; use **startup()** for loading and **search()** with **matcher** for fuzzy matching.

---

## Key bindings (footer)

Key bindings (e.g. `q` for quit) are separate from the command palette. They are declared on the App (or widget) and shown in the footer. See [04-exit-and-shutdown.md](04-exit-and-shutdown.md) for quit bindings. The command palette is for **discoverable** actions; bindings are for **fast** actions.

Next: [03-menus-and-traversal.md](03-menus-and-traversal.md) for using and traversing the command palette and menus.
