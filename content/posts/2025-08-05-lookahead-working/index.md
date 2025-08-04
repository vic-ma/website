+++
title      = "It's alive!"
date       = '2025-08-05'
slug       = 'gsoc-6'
categories = ['GSoC']
+++

In the last two weeks, I've been working on my lookahead-based word suggestion algorithm. And it's finally functional! There's still a lot more work to be done, but it's great to see that the [original problem](http://localhost:1313/posts/gsoc-5/#design-doc) I set out to solve is now solved by my new algorithm.


## Without my changes

Here's what the upstream Crosswords Editor looks like, with a problematic grid:

![Broken behaviour](https://victorma.ca/posts/gsoc-6/broken.png)

The editor suggests words like *WORD* and *WORM*, for the 4-Across slot. But none of the suggestions are valid, because the grid is actually unfillable. This means that there are no possible word suggestions for the grid.

The words that the editor suggests do work for 4-Across. But they do not work for 4-Down. They all cause 4-Down to become a nonsensical word.

The problem here is that the current word suggestion algorithm only looks at the row and column where the cursor is. So it sees 4-Across and 1-Down---but it has no idea about 4-Down. If it could see 4-Down, then it would realise that no word that fits in 4-Across also fits in 4-Down---and it would return an empty word suggestion list.


## With my changes

My algorithm fixes the problem by considering *every* intersecting slot of the current slot. In the example grid, the current slot is 4-Across. So, my algorithm looks at 1-Down, 2-Down, 3-Down, and 4-Down. When it reaches 4-Down, it sees that no letter fits in the empty cell. Every possible letter leads to either 4-Across or 4-Down or both slots to contain an invalid word. So, my algorithm correctly returns an empty list of word suggestions.

![Fixed behaviour](https://victorma.ca/posts/gsoc-6/fixed.png)