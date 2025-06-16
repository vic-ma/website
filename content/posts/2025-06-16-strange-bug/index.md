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

But that's not what actually happens. In reality, the word suggestions list changes quite dramatically. The order of the words definitely changes. And it looks like there may even be words in one list that are missing in another. What's going on here?


## Understanding the code

My first step was to understand how the code for the word suggestions list works. This is what I spent the first week on. I took [notes](https://pad.gnome.org/s/R5IvXtNwS#Intersection-code-notes) along the way, in order to solidify my understanding, and to serve as a useful reference in the future. I especially found it helpful to create diagrams for the word list resource (a pre-compiled resource that the code uses).

![Word list resource diagram](https://victorma.ca/posts/strange-bug/diagram.png)

By the end of the first week, I had a good idea of how the word-suggestions-list code works. The next step was to figure out what was causing the bug.


## Investigating the bug

After doing some testing, I realized that the seemingly random ordering that the bug caused is not so random after all! The lists are actually all in alphabetical order---but based on the letter that corresponds to the cell.

What I mean is this:
* The word suggestions list for cell `α` is sorted alphabetically by the first letter of the words. (This is normal alphabetical order.) For example:
  ```
  ALE, AXE, BAY, BOA, CAB
  ```
* The word suggestions list for cell `β` is sorted alphabetically by the second letter of the words. For example:
  ```
  CAB, BAY, ALE, BOA, AXE
  ```
* The word suggestions list for cell `γ` is sorted alphabetically by the third letter of the words. For example:
  ```
  BOA, CAB, ALE, AXE, BAY
  ```


## Fixing the bug

The cause of the bug is quite simple: The function that generates the word suggestions list does not sort the list before it returns it. So the order of the list is whatever order we added the words in. And because of how our implementation works, that order happens to be alphabetical, based on the letter that corresponds to the cell.

The fix for the bug is also quite simple---at least theoretically. All we need to do is sort the list before we return it. In actuality, this fix runs into some other problems that need to be addressed. But anyway, that's for me to work on this week.