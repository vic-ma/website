+++
title      = 'My first design doc'
date       = '2025-07-15'
slug       = 'gsoc-5'
categories = ['GSoC']
+++

In the last two weeks, I investigated some bugs, tested some fonts, and started working on a design doc.


## Bugs

I found two more UI-related bugs ([1](https://gitlab.gnome.org/jrb/crosswords/-/issues/280), [2](https://gitlab.gnome.org/jrb/crosswords/-/issues/282)). These are in addition to the ones I mentioned in my last blog post---and they're all related. They have to do with GTK and sidebars and resizing.

I looked into them briefly, but in the end, my mentor decided that the bugs are complicated enough that he should [handle them himself](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/258). His fix was to replace all the `.ui` files with [Blueprint](https://gitlab.gnome.org/GNOME/blueprint-compiler) files, and then make changes from there to squash all the bugs. The port to Blueprint also makes it much easier to edit the UI in the future.


## Font testing

Currently, GNOME Crosswords uses the default GNOME font, Cantarell. But we've never really explored the possibility of using other fonts. For example, what would Crosswords look like with a monospace font? Or with a handwriting font? This is what I set out to discover.

To change the font, I used GTK Inspector, combined with this CSS selector, which targets the grid and word suggestions list:
```
edit-grid, wordlist {
    font-family: FONT;
}
```
This let me dynamically change the font, without having to recompile each time. I created a document with all the [fonts that I tried](https://pad.gnome.org/s/6mTne5Ehs).

Here's what *Source Code Pro*, a monospace font, looks like. It gives a more rigid look---especially for the word suggestion list, where all the letters line up vertically.
![Monospace font](https://victorma.ca/posts/gsoc-5/monospace.png)

And here's what *Annie Use Your Telescope*, a handwriting font, looks like. It gives a fun, charming look to the crossword grid---like it's been filled out by hand. It's a bit too unconventional to use as the default font, but it would definitely be cool to add as an option that the user can enable.
![Handwriting font](https://victorma.ca/posts/gsoc-5/handwriting.png)


## Design doc

My current task is to improve the word suggestion algorithm for the Crosswords Editor. Last week, I starting working on a [design doc](https://pad.gnome.org/s/OAL239g-o) that explains my intended change. Here's a short snippet from the doc, which highlights the problem with our current word suggestion algorithm:

> Consider the following grid:
> ```
> +---+---+---+---+
> |   |   |   | Z |
> +---+---+---+---+
> |   |   |   | E |
> +---+---+---+---+
> |   |   |   | R |
> +---+---+---+---+
> | W | O | R |   | < current slot
> +---+---+---+---+
> ```
> The 4-Down slot begins with *ZER*, so the only word it can be is *ZERO*. This means that the cell in the bottom-right corner must be the letter *O*.
>
> But 4-Across starts with *WOR*. And *WORO* is not a word. So the bottom-right corner cannot actually be the letter *O*. This means that the slot is unfillable.
>
> If the cursor is on the bottom right cell, then our word suggestion algorithm correctly recognizes that the slot is unfillable and returns an empty list.
>
> But suppose the cursor is on one of the other cells in 4-Across. Then, the algorithm has no idea about 4-Down and the constraint it imposes. So, the algorithm returns all words that match the filter *WOR?*, like *WORD* and *WORM*---even though they do not actually fit the slot.

### CSPs

In the process of writing the doc, I came across the concept of a [constraint satisfaction problem (CSP)](https://cs.uwaterloo.ca/~jhoey/teaching/cs486/lecture4-nup.pdf), and the related AC-3 algorithm. A CSP is a formalization of a problem that...well...involves satisfying a constraint. And the AC-3 algorithm is an algorithm that's sometimes used when solving CSPs.

The problem of filling a crossword grid can be formulated as a CSP. And we can use the AC-3 algorithm to generate perfect word suggestion lists for every cell.

This isn't the approach I will be taking. However, we may decide to implement it in the future. So, I documented the [AC-3 approach](https://pad.gnome.org/s/OAL239g-o#Grid-Level-algorithm) in my design doc.