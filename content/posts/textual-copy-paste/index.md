+++
title = 'Copying and pasting in Textual'
date = 2024-10-02T22:36:57+01:00
draft = true
author = "Darren Burns"
+++

Terminal emulators are in general quite limited when it comes to copying and pasting. Fortunately, when writing Textual apps, we have a few options available to us.

Textual has some built-in support for copy-paste; *but* it's dependent on the terminal your app is running in.

## Textual's built-in support

Textual's `App` class has a method called `copy_to_clipboard(text: str)`. Pass a string into it, and that string will be placed onto the clipboard of the machine your terminal emulator is running on (*if* your emulator supports it).

Unfortunately, if the user of the app is running in MacOS Terminal.app or another unsupported terminal, nothing will happen.

## Consider `pyperclip`

If you need broader clipboard support you can try `pyperclip`. `pyperclip` lets you add support for copying and pasting text to/from the clipboard in a cross-platform manner. It's important to note however, that it'll copy to the clipboard of the machine that the app is running in. This is an important distinction, as it means if your app is running over SSH then it'll copy to the clipboard of the remote host, rather than your local machine!

## Integrating `pyperclip` with Textual

Let's look at an example of how we might create a read-only `TextArea`, which we can copy some text from. The example assumes you've installed `pyperclip` into your environment.

### Subclassing `TextArea` and adding a new binding

First, we subclass `TextArea`, and add a binding for which will call the `action_copy_selection` method:

```python
class ReadOnlyTextArea(TextArea):
    BINDINGS = [Binding("y,c", action="copy_selection", description="Copy selection")]
    
    def on_mount(self) -> None:
        self.read_only = True

    def action_copy_selection(self):
        """Copy to the clipboard of the machine the app is running on"""
```

This links the `y` and `c` keys on your keyboard to the `action_copy_selection` method. On pressing one of those keys, `action_copy_selection` will be called, so that's where we'll put our code.

### Implementing the `action_copy_selection` method

Note that when we use `pyperclip`, we also need some special handling for the case where the user's system has no clipboard management utility (such as Ubuntu by default). If this is the case, you can choose to alert the user however you wish. The inability to copy some text generally wouldn't be considered a *fatal* error, so I like to simply inform the user using a toast rather than crashing their app!

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

Putting the code above into our `action_copy_selection` method means that whenever we press `y` on our keyboard, that text will be copied and the user will be informed via a toast popup.

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

If you run the app and select some text using the mouse or by holding shift and moving the cursor, you should see a toast popup indicating that the text has been copied.

Switch over to another application and try pasting to confirm that it works.

## Having issues?

If you're having issues with clipboard functionality, consider the following:

1. Ensure pyperclip is installed correctly in your environment.
2. On Linux, make sure you have a clipboard utility installed (e.g. `xclip` or `xsel`).
3. If using SSH, remember that pyperclip will copy to the remote machine's clipboard.

## So which should I use?

If you're distributing your app, `pyperclip` is often going to be the better choice, as it has wider support. However, if your app might reasonably be used over SSH, you might consider allowing *both*.

You may wish to override `App.copy_to_clipboard` with a `pyperclip` version, or have it be configurable via a file or environment variable. You could choose to have different keybindings for "copy via terminal" and "copy via host." Or, you may wish to try some smarter detection and choose which one to used based on the user's environment.

```python
from textual.app import App

class MyApp(App[None]):
    def copy_to_clipboard(self, text: str) -> None:
        """Implement your own custom logic if needed."""
```

Ultimately, the choice is yours and depends on your application and its users!
