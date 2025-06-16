+++
title      = 'A strange bug'
date       = '2025-06-16'
slug       = 'strange-bug'
categories = ['GSoC']
draft      = true
+++


In the last two weeks, I've been trying to fix a [strange bug](https://gitlab.gnome.org/jrb/crosswords/-/issues/269) that causes the word suggestions list to have the wrong order sometimes.

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

You should expect the word suggestions list to stay the same, regardless of which cell your cursor is on. After all, all three cells have the exact same information: that the 1-Across slot is empty, and the intersecting vertical slot of whatever cell we're on (1-Down, 2-Down, or 3-Down) is also empty.

There are no restrictions whatsoever, so all three cells should have a word suggestion list that includes every three-letter word in the word list.

But that's not what actually happens. In reality, the word suggestions list appears to change quite dramatically. At the very least, the order of the words changes. And it seems like some words may even appear in one list but not another. What's going on here?


## Understanding the code

My first step was to understand how the code for the word suggestions list works. This is what I spent the first week on. I took [notes](https://pad.gnome.org/s/R5IvXtNwS#Intersection-code-notes) along the way, in order to solidify my understanding, and to serve as a useful reference in the future. I especially found it helpful to create diagrams for the word list resource (a pre-compiled resource that the code uses).

![Word list resource diagram](https://s3.us-east-2.amazonaws.com/hedgedoc-gnome-org/uploads/0f1b4663-f209-4f39-8630-4a3ecd7b021a.png)

By the end of the first week, I had a good idea of how the intersection code worked. The next step was to figure out what was causing the bug and how to fix it.

## Investigating the bug

After doing some testing, I figured out what was going on with the word suggestion lists. There was in fact a pattern to the seemingly random order.