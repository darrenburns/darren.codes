+++
title = "Textual's utility containers"
date = 2024-10-02T21:46:31+01:00
draft = false
author = "Darren Burns"
+++

I almost always favour using [Textual's utility containers](https://textual.textualize.io/guide/layout/#utility-containers) over CSS for layout.

When using utility containers, we can look at a widget's `compose` method and know immediately how it will be displayed, without having to manually search
for the relevant CSS styles.

When deciding which container to use, you must ask yourself:

1. How do I want the content to flow? Vertically or horizontally?
2. If there's more content than available space, do I want to be able to scroll?

Based on your answers, you'll reach for one of `Vertical`, `Horizontal`, `VerticalScroll` or `HorizontalScroll`. I use these containers *extensively*, and I think if you're finding yourself writing CSS like "`layout: vertical`", then you ought to try out the container approach instead. I tend to avoid using grid layout, as I find it harder to reason about and maintain.

## Scrollable containers are focusable

One gotcha with scrollable containers is that they're *focusable* by default. I almost always set `can_focus=False` though, because:

1. **Textual will scroll on focus anyway** - focusing a child widget in the container will cause it to scroll into view anyway.
2. **Scrolling will almost always work anyway** - if a child widget of the container is focused, unless that child consumes the keys used to scroll, scrolling will still work since scroll events bubble up the DOM.
3. **No default styling** - although they're focusable by default, there's no visual feedback which hints that a scrollable container is focused. Without additional CSS, it's difficult to track where focus has gone to when one of these containers receives focus.

Here's an example of a scrollable container which has `can_focus=False`:

```python
with VerticalScroll() as vs:
	vs.can_focus = False
	yield Label("Do NOT press the button below")
	yield Button("Press me!")
```

However, if your container contains only *unfocusable* children, then you'll probably want to keep `can_focus=True`, otherwise you may find you have no means of scrolling.
