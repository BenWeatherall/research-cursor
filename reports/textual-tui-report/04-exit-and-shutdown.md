# 04 — Exit and shutdown

This document describes how to **exit** a Textual app in a consistent way: the built-in Quit command, key bindings, and programmatic exit. It also notes shutdown and cleanup.

References: [Command palette](https://textual.textualize.io/guide/command_palette/) (Quit command), [Footer / BINDINGS](https://textual.textualize.io/widgets/footer/), [App.exit](https://textual.textualize.io/api/app/).

---

## Built-in Quit command

Textual provides a **Quit** system command by default. It is available in the command palette (Ctrl+P): user types (e.g. "qu"), selects "Quit", presses Enter, and the app exits. No code is required beyond using the default `get_system_commands()` from the base App.

---

## Key binding for quit

For quick exit, bind a key (e.g. **q**) to the **quit** action. Declare **BINDINGS** on the App; the footer will show the binding. Use the **action** name `"quit"` so Textual performs a clean exit.

```python
from __future__ import annotations

from textual.app import App, ComposeResult
from textual.binding import Binding
from textual.widgets import Static


class QuitBindingApp(App[None]):
    BINDINGS = [
        Binding("q", "quit", "Quit"),
    ]

    def compose(self) -> ComposeResult:
        yield Static("Press 'q' to quit.")
```

- **Binding(key, action, description)**: `action="quit"` triggers the same shutdown path as the Quit command.
- **description**: Shown in the footer (e.g. "Quit").

---

## Programmatic exit

To exit from code (e.g. after saving or when a dialog is confirmed), call **`self.exit()`** (or **`app.exit()`**). You can pass a result value if your App is generic, e.g. `App[str]`:

```python
class ResultApp(App[str]):
    def do_confirm_exit(self) -> None:
        self.exit("saved")

# When run:
# result = app.run()
# assert result == "saved"
```

For a simple exit with no return value, call `self.exit()` with no arguments (or `self.exit(None)`).

---

## Shutdown and cleanup

- **Exit path**: Quit command, quit binding, or `self.exit()` all lead to the same shutdown: message loop stops, app exits.
- **Cleanup**: Override **`on_unmount()`** on the App (or on widgets/screens) to release resources when the app or widget is torn down. Avoid long-running work in `on_unmount()`; keep it to releasing handles, closing files, etc.

```python
def on_unmount(self) -> None:
    if self._resource:
        self._resource.close()
```

---

## Consistent exit behaviour (summary)

| Method | Use case |
|--------|----------|
| **Quit command** (palette) | Discoverable; user presses Ctrl+P, searches "quit", Enter. |
| **Key binding** `action="quit"` | Fast; e.g. `q` or `ctrl+q`. |
| **`self.exit()`** | Programmatic; e.g. after "Save and quit" or error. |

Recommendation: support **both** the Quit command (default) and a **key binding** (e.g. `q`) so power users can exit quickly and new users can find Quit in the palette.

---

## Minimal runnable example

```python
from __future__ import annotations

from textual.app import App, ComposeResult
from textual.binding import Binding
from textual.widgets import Static


class ExitApp(App[None]):
    BINDINGS = [Binding("q", "quit", "Quit")]

    def compose(self) -> ComposeResult:
        yield Static("Press 'q' or use Ctrl+P → Quit to exit.")

if __name__ == "__main__":
    ExitApp().run()
```

Next: [05-cli-integration.md](05-cli-integration.md) for integrating the TUI with argparse, Click, or Typer.
