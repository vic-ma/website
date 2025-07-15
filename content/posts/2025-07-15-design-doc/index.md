+++
title      = 'My first design doc'
date       = '2024-07-14'
slug       = 'gsoc-5'
categories = ['GSoC']
+++

In the last two weeks, I investigated some bugs, tested some fonts, and started working on a design doc.


## Bugs

I found a two more UI-related bugs ([1](https://gitlab.gnome.org/jrb/crosswords/-/issues/280), [2](https://gitlab.gnome.org/jrb/crosswords/-/issues/282)). These are in addition to the ones I mentioned in my last blog post---and they're all connected. They have to do with GTK and sidebars and resizing, and other things like that. 

Anyway, I looked into them briefly, but in the end, my mentor decided that the bugs are complicated enough that he should [handle them himself](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/258). His fix involves replacing all the `.ui` files with [Blueprint](https://gitlab.gnome.org/GNOME/blueprint-compiler) files. This will make it much nicer to make changes to the UI in the future.


## Font testing

Currently, GNOME Crosswords uses the default GNOME font, Cantarell. But we've never really explored the possibility of using other fonts. For example, what would Crosswords look like with a monospace font? Or with a handwriting font? That's what I set out to discover.

To change the font, I used GTK Inspector, combined with this CSS selector, which targets the grid and word suggestions list:
```
edit-grid, wordlist {
    font-family: FONT;
}
```
Then, I could dynamically change the font to whatever I wanted.

Here's what *Source Code Pro*, a monospace font, looks like:
![Monospace font](https://victorma.ca/posts/gsoc-5/monospace.png)

And here's what *Annie Use Your Telescope*, a handwriting font, looks like:
![Handwriting font](https://victorma.ca/posts/gsoc-5/handwriting.png)