# 04 — TUI (Textual)

This document covers the **Textual** TUI in `tui/`: how to define the App, use `compose()` and optional screens, and how the entry point invokes it. Use Textual for the TUI path only; the CLI uses Typer.

---

## Textual App in `tui/`

Define your main **App** in `tui/app.py` (or `tui/__init__.py`). Subclass `textual.app.App`, override `compose()` to declare the initial widget tree, and use type hints. The App is the root of the TUI; it should call into **core** for behaviour, not duplicate logic.

```python
"""TUI implemented with Textual."""
from __future__ import annotations

from textual.app import App, ComposeResult
from textual.widgets import Header, Static


class MyApp(App[None]):
    """Main TUI application."""

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static("Hello, TUI.", id="main")
```

- **`App[None]`**: Generic indicates the type of the value returned when the app exits (use `App[str]` if you exit with a result).
- **`compose()`**: Yields widgets in order; Textual adds them to the default screen.

---

## Invocation from the entry point

The entry point (`__main__.py`) runs the TUI when **no arguments** are passed (default). It **instantiates** your App and calls **`run()`** so that Textual drives the event loop. Run via `python -m my_tool` with no args so the TUI launches; the entry point checks `sys.argv` to decide.

```python
# In __main__.py (TUI branch):
from my_tool.tui.app import MyApp
app = MyApp()
app.run()
```

Do not rely on `textual run my_tool.tui.app` if you need to pass arguments (e.g. `--tui`) into your app; the module entry point gives you control over argv.

---

## Compose and screens

- **`compose()`**: Declarative; yield widgets (e.g. `Header`, `Static`, containers like `VerticalScroll`). Use `id="..."` to query widgets later with `self.query_one("#id", WidgetType)`.
- **Screens:** For multiple full-size views, define separate `Screen` classes and use `push_screen()` / `pop_screen()`; only one screen is active at a time.

Example with a container:

```python
from textual.containers import VerticalScroll

def compose(self) -> ComposeResult:
    yield Header()
    with VerticalScroll():
        yield Static("Content", id="main")
```

See the project’s **Textual TUI report** (e.g. in `reports/textual-tui-report/`) for: App and Screen structure, commands and dynamic options, menus and traversal, and exit behaviour.

---

## Calling core from the TUI

When the user triggers an action in the TUI (e.g. “Run job”), the handler should call the **same core functions** the CLI uses. Catch core exceptions and show errors in the UI (e.g. a dialog or banner), instead of printing or exiting.

```python
def on_button_pressed(self) -> None:
    try:
        from my_tool.core.service import run_job, CoreError
        result = run_job(self.job_id, config={})
        self.notify(result)
    except CoreError as e:
        self.notify(str(e), severity="error")
```

---

## Summary

- **Placement:** Textual App in `tui/app.py`; core in `core/`.
- **Invocation:** Entry point instantiates `MyApp()` and calls `app.run()` in TUI mode.
- **Style:** Use `compose()`, type hints, and optional screens; call core from handlers; show errors in the UI.

See also: [01 — Entry point and structure](01-entry-point-and-structure.md) | [02 — Shared core](02-shared-core.md) | [03 — CLI (Typer)](03-cli-typer.md) | [Index](index.md)
