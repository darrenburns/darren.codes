+++
title = 'Copying and pasting in Textual'
date = 2024-10-02T22:36:57+01:00
draft = false
author = "Darren Burns"
tags = ['textual', 'python']
+++

Terminal emulators are in general quite limited when it comes to copying and pasting.
Fortunately, we have a few options when writing Textual apps which allow us to add support for copy and paste.
Each has its own trade-offs.
Let's explore these options and look at how we can integrate them into our apps.

## `App.copy_to_clipboard`

Textual's `App` class has a method called [`copy_to_clipboard`](https://textual.textualize.io/api/app/#textual.app.App.copy_to_clipboard).
Pass a string into it, and it gets copied to the clipboard on the machine the emulator is running on.
There is a caveat here: If the terminal doesn't support the required protocol then nothing will happen!

### Aside: How does it work?

`App.copy_to_clipboard` uses the [OSC 52 ANSI escape sequence](https://en.wikipedia.org/wiki/ANSI_escape_code#OSC_(Operating_System_Command)_sequences) to tell the terminal emulator to copy text to the system clipboard.
The terminal emulator doesn't consider the *source* of the text, it justs sends it to the system clipboard.
Running an application inside tmux over SSH? The text can still be copied to your local system's clipboard!

This sequence has a reasonable degree of support, with MacOS terminal being the notable exception.
If a terminal supports OSC 52, then `App.copy_to_clipboard` will work.

Here's a table showing OSC 52 support for common terminal emulators:

| Terminal Emulator | OSC 52 Support |
|-------------------|----------------|
| iTerm2            | Yes            |
| Kitty             | Yes            |
| Alacritty         | Yes            |
| Windows Terminal  | Yes            |
| WezTerm           | Yes            |
| Ghostty           | Yes            |
| Gnome Terminal    | No             |
| Konsole           | No (Coming soon) |
| MacOS Terminal    | No             |


## Consider `pyperclip`

If you need broader clipboard support you can try `pyperclip`. `pyperclip` is a cross-platform library for interacting with the clipboard.

It's important to note that `pyperclip` copies to the clipboard of the machine that the *app* is running on.
This is an important distinction.
If your app is running over SSH, `pyperclip` copies to the clipboard of the *remote host*!

## Integrating `pyperclip` with Textual

Let's look at an example of how we can create a read-only `TextArea`, which we can copy some text from.
The example assumes you've installed `pyperclip` into your environment.

### Subclassing `TextArea` and adding a new binding

First, subclass `TextArea`, and add a binding to call the `action_copy_selection` method:

```python
class ReadOnlyTextArea(TextArea):
    BINDINGS = [Binding("y,c", action="copy_selection", description="Copy selection")]
    
    def on_mount(self) -> None:
        self.read_only = True

    def action_copy_selection(self):
        """Copy to the clipboard of the machine the app is running on"""
```

This links the `y` and `c` keys on your keyboard to the `action_copy_selection` method.
Press one of those keys, and `action_copy_selection` runs.
Let's implement that method now.

### Implementing the `action_copy_selection` method

The snippet below implements `action_copy_selection` using `pyperclip`.
It gets the `selected_text` from the `TextArea`, then tries to import and call `pyperclip`.
There is a gotcha here: clipboard functionality is not guaranteed.
On Ubuntu, the clipboard is provided by `xclip` or `xsel`.
If these are unavailable, `pyperclip` will raise an exception which we must handle.
Lack of clipboard support shouldn't be a fatal error, so graceful handling is important.
I opted to notify the user about the problem using `self.notify`, which shows a toast popup at the bottom right of the screen.

```python
    def action_copy_selection(self):
        """Copy to the clipboard of the machine the app is running on"""
        text_to_copy = self.selected_text
        try:
            import pyperclip

            pyperclip.copy(text_to_copy)
        except pyperclip.PyperclipException as exc:
            # Show a toast popup if we fail to copy.
            self.notify(
                str(exc),
                title="Clipboard error",
                severity="error",
            )
        else:
            self.notify(f"Copied {len(text_to_copy)} characters!", title="Copied selection")
```

With our action method in place, we can now select text and press `y` to copy it to the clipboard.

### Using our new widget

Here's a simple Textual app that demonstrates the use of our `ReadOnlyTextArea`:

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer

class ClipboardDemoApp(App):
    def compose(self) -> ComposeResult:
        yield Header()
        yield ReadOnlyTextArea("Select this text and press 'y' to copy!")
        yield Footer()

if __name__ == "__main__":
    app = ClipboardDemoApp()
    app.run()
```

If you run the app and select some text using the mouse or by holding shift and moving the cursor, you'll see a toast popup indicating that the text was copied.

Switch over to another application and try pasting to confirm that it works.

## Which should I use?

If you're distributing your app, `pyperclip` is likely the better choice due to broader support.
However, if your app might reasonably be used over SSH, you may consider supporting *both*.

You could override `App.copy_to_clipboard` with a `pyperclip` version, or make it configurable.
You could choose to have different keybindings for "copy via terminal" and "copy via host."

Another option is to try some smarter detection and choose which one to used based on the user's environment.
For example, if the user is using the default MacOS terminal (which lacks support for copy/paste), you may opt to fall-back to `pyperclip`.

```python
from textual.app import App

class MyApp(App[None]):
    def copy_to_clipboard(self, text: str) -> None:
        """Copy text to the clipboard"""
        is_apple_terminal = os.environ.get("TERM_PROGRAM", "") != "Apple_Terminal"
        if is_apple_terminal:
            self.copy_with_pyperclip(text)
        else:
            super().copy_to_clipboard(text)

    def copy_with_pyperclip(self, text: str) -> None:
        """Copy text to the clipboard using pyperclip"""
        # don't forget error-handling!
        import pyperclip
        pyperclip.copy(text)
```

This is of course rather naive, as `TERM_PROGRAM` alone does not provide a complete picture of the user's environment.
A more robust solution might check the XTerm version, [like Emacs does](https://github.com/emacs-mirror/emacs/blob/f18af6cd5cb7dbbf7420ec2d3efed4e202c4f0dd/lisp/term/xterm.el#L713).
