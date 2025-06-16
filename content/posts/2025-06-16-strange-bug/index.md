+++
title      = 'A strange bug'
date       = '2025-06-16'
slug       = 'strange-bug'
categories = ['GSoC']
draft      = true
+++

In the last two weeks, I've been trying to fix a [strange bug](https://gitlab.gnome.org/jrb/crosswords/-/issues/269). This bug causes the word suggestions list to appear to differ, for cells that should have the same word suggestions list.

For example, suppose you have an empty 3x3 grid. Now suppose that you move your cursor across the cells of the 1-Across slot (labelled `α`, `β`, and `γ`). 
```
+---+---+---+
| α | β | γ |
+---+---+---+
|   |   |   |
+---+---+---+
|   |   |   |
+---+---+---+
```

You should expect the word suggestions list to stay the same, regardless of which cell your cursor is on. After all, all three cells have the exact same information: that the 1-Across slot is empty, and so is the intersecting vertical slot of whatever cell we're on (1-Down, 2-Down, or 3-Down).

But that's not what actually happens. In reality, the word suggestions list appears to change dramatically, at least in order, and perhaps also in content.