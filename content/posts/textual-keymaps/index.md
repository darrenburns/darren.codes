+++
title = 'Custom keymaps in Textual'
date = 2024-10-03T10:42:09+01:00
draft = false
author = "Darren Burns"
tags = ['textual', 'python', 'tutorial']
+++

[Textual 0.82.0](https://github.com/Textualize/textual/releases/tag/v0.82.0) adds support for *keymaps*, which make it easy to customise keybindings based on config or user preferences.

Keymaps allow you to swap the keys associated with a `Binding` at runtime with a single method call.

- You could switch to a Vim keymap if a user of your app enables "Vim mode".
- You could read a keymap from a file on the users drive and swap it in on app startup.
- You could swap the increment and decrement keys on April 1st!

The possibilities are endless!

Keymaps are simple to use - just give an `id` to any `Binding` you wish to permit overrides for, then call the `App.set_keymap` method when your app starts, passing in a dict mapping that binding ID to the new key.

## Vim keys

Here's an example of how you'd define a binding with an overridable key:

```python
from textual.binding import Binding
from textual.reactive import reactive
from textual.widgets import Static

class Counter(Static, can_focus=True):
    BINDINGS = [
        # The id is a string used to identify this binding in the keymap.
        # You may wish to namespace them to make managing them easier!
        Binding("up", "change_count(1)", "Increment", id="counter.increment"),
        Binding("down", "change_count(-1)", "Decrement", id="counter.decrement"),
    ]

	count = reactive(0)
    """Stores the current value of the counter."""

	def action_change_count(self, delta: int) -> None:
        """Change the counter by delta."""
	    self.count += delta
```

Pressing `up` or `down` on your keyboard will increment or decrement the counter.

Since we've given these bindings IDs, they can now be *overridden* by a keymap.
Let's override these two bindings so that we can also use Vim keys to change the counter.

When our `App` launches, we just need call `App.set_keymap` to specify our overrides.
A good place to do this would be in `App.on_mount`:

```python
from textual.app import App

class MyCounterApp(App[None]):
    def on_mount(self) -> None:
	    vim_keymap = {
	        "counter.increment": "k,up",
	        "counter.decrement": "j,down",
	    }
        self.set_keymap(vim_keymap)

```

Now users will be able to use `k` *or* `up` to increment the counter, and `j` or `down` to decrement it!

**It's important to note that if we still wish for the `up` and `down` keys to work, _we need to include them in the keymap too!_**

That's all there is to it!

Keymaps should make it easier for developers to support users who want to customise their keybindings, and reduce the impact of a default 
keybinding choice clashing with the terminal emulator or multiplexer.

Check out the API documentation for keymaps [here](https://textual.textualize.io/api/app/#textual.app.App.set_keymap)!
