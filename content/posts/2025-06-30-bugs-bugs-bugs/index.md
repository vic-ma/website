+++
title      = 'Bugs, bugs, and more bugs!'
date       = '2025-07-01'
slug       = 'bugs-bugs-bugs'
categories = ['GSoC']
+++

In the past two weeks, I worked on two things:
* Squashing a rebus bug.
* Combine the two suggested words lists into one.


## The rebus bug

A rebus cell is a cell that contains more than one letter in it. These aren't too common in crossword puzzles, but they do appear occasionally---and especially so in harder puzzles.

Our word suggestions lists were not working for slots with rebus cells. More specifically, if the cursor was on a cell that's within `letters in rebus - 1` cells to the right of a rebus cell, then an assertion would fail, and the word suggestions list would be empty.

The cause of this bug is that our intersection code (which is what generates the suggested words) was not accounting for rebuses at all! The fix was to modify the intersection code to correctly count the additional letters that a rebus cell contains.


## Combine the suggested words lists

The Crosswords editor shows a the words list for both Across and Down, at the same time. This is different from what most other crossword editors do, which is to have a single suggested words list that switches between Across and Down, based on the cursor's direction.

I think having a single list is better, because it's visually cleaner, and you don't have to take a second to find right list. It also so happens that we have a problem with our sidebar jumping, in large part because of the two suggested words lists.

So, we decided that I should combine the two lists into one. To do this, I removed the second list widget and list model, and then I added some code to change the contents of the list model whenever the cursor direction changes.


## More bugs!

I only started working on the rebus bug because I was working on the word suggestions bug. And I only started working on that bug because I discovered it while using the Editor. And it's a similar story with the words lists unification task. I only started working on it because I noticed the sidebar jumping bug.

Now, the plan was that after I fixed those two bugs, I would turn my attention to a bigger task: adding a step of lookahead to our fill algorithm. But alas, as I was fixing the two bugs, I noticed a few more bugs. But they shouldn't take too long, and they ought to be fixed. So I'm going to do that first, and then transition to working on the fill lookahead task.