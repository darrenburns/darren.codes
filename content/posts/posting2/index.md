+++
title = 'Introducing Posting 2.0'
date = 2024-10-19T13:40:01+01:00
draft = false
author = "Darren Burns"
tags = ['posting', 'textual', 'projects']
+++

Posting is a powerful HTTP client that runs entirely inside your terminal.

Version 2.0 has just arrived, bringing some powerful new features to Posting, including scripting, keymaps, and more!

{{< figure src="posting-screenshot.svg" alt="posting-2" title="Posting 2.0 - a TUI HTTP client" >}}

## Pre-request and post-response scripts

A common feature request has been around adding scripting capabilities to Posting.
With 2.0, you can now attach Python functions to requests, and have them run before or after the request is sent.
All output from these scripts is captured and displayed in the new "Scripts" tab.
This makes it easy to see the results of your script executions and troubleshoot any issues.

{{< figure src="script-output.png" alt="Script output" title="Viewing the status and output of scripts in Posting's new 'Scripts' tab." >}}

This is an extremely versatile feature.
You can use scripts to perform setup, set variables which requests and other scripts can access, modify outgoing requests, and much more.
Here's an example of some Python code which you can attach to a request:

```python
def on_request(request: RequestModel, posting: Posting) -> None:
    request.headers.append(Header(name="X-Custom-Header", value="foo"))
    request.auth = Auth.basic_auth("username", "password")
    print("Request is being sent!")
    posting.notify("Request is being sent!")
```

The API is currently a bit rough around the edges, but it'll be improved in future releases.

Attached scripts can even be edited in an external editor, and Posting will automatically reload them when they are changed.

{{< figure src="auto-reload.png" alt="auto-reload" title="Automatic reloading" >}}

There's also a convenience keyboard shortcut which allows you to quickly open a request's scripts in your preferred `$EDITOR`,
so you could quickly swap over to `vim` or `nano` to make changes, then return to Posting to continue working.

Check out the [scripting docs](https://posting.sh/guide/scripting) for more details!

## Keymaps

Posting 2.0 also allows you to declare custom keybindings, thanks to an update in the underlying [Textual](https://textual.textualize.io/) framework.
For 2.0 only "global" keybindings can be remapped, but this will be expanded in future releases to include most keybindings.

You can customise your keymap by adding a `keymap` section to your Posting config.
Here's a short example which changes the default for sending the request to `ctrl+r`:

```yaml
keymap:
  send-request: "ctrl+r"
```

Hopefully this will help you feel more at home in Posting, and avoid awkward clashes with emulators and multiplexers!

See the [keymap docs](https://posting.sh/guide/keymap) for more details!

## Customisable hostnames

Posting is a TUI, which means it works great over SSH.
However, when working inside a TUI, it's sometimes easy to forget which host you're on.

In 2.0, there's a new config option which allows you to customise the hostnames which are displayed in the header, including styling them with Rich markup to make them stand out.

This is useful if you work with Posting in multiple environments, such as your local machine, a staging server, and a production server.

{{< figure src="header-local.png" alt="header-local" title="Running Posting locally" >}}

{{< figure src="production-blink.gif" alt="header-production" title="Running Posting on a production server" >}}

Here's the config I use on my "production" server to achieve the effect above:

```yaml
heading:
  hostname: "[black on #ff0000 bold blink]PRODUCTION[/]"
```

## Documentation

Posting's documentation has been expanded, making it easier than ever to get started!

The additions include:

- Duplicating and "quick duplicating" requests.
- [Scripting](https://posting.sh/guide/scripting).
- [Keymaps](https://posting.sh/guide/keymap).
- [Integrating Posting with external tools](https://posting.sh/guide/external_tools).
- An improved [getting started guide](https://posting.sh/guide/) which better showcases some quality-of-life features that may not be obvious at first glance :)

## What's next?

Textual 1.0 is around the corner, and will unlock a better theming system for Posting as well as a more powerful command palette!

I've also braindumped some features I'd like to implement in Posting [here](https://posting.sh/roadmap).

The new features mentioned in this post and those on the roadmap are very much driven by users.
If you'd like to suggest new features, please open a [discussion on GitHub](https://github.com/darrenburns/posting/discussions)!
