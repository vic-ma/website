+++
title      = 'Bugs, bugs, and more bugs!'
date       = '2025-07-01'
slug       = 'bugs-bugs-bugs'
categories = ['GSoC']
draft      = true
+++

In the past two weeks, I worked on two things:
* Squashing a rebus bug.
* Unifying the two suggested words lists into one.


## The rebus bug

A rebus cell is a cell that contains more than one letter. These aren't too common in crossword puzzles, but they do appear occasionally---especially in harder puzzles.

The bug was that our word suggestions lists were not working for slots with rebus cells in them. More specifically, if the cursor was on a cell that's within `letters in rebus - 1` cells to the right of a rebus cell, then an assertion would fail, and the word suggestions list would be empty.

Anyway, the cause of this bug is that our intersection code (which is what generates the suggested words) was not accounting for rebuses at all! The fix for this was to modify the logic to account for the additional letters that a rebus cell contains.


## Unify the suggested word lists

The Crosswords editor shows a the words list for both Across and Down, at the same time. This is different from what other crossword editors do, which is to have a single suggested words list that switches between Across and Down, based on the cursor's direction.

I think having a single list is better, because it's visually cleaner, and you don't have to take a second to look at the right list. It also so happens that we have a problem with our sidebar jumping, in large part because of the two suggested words lists.

So, we decided that I should unify the two lists into one. To do this, I removed the second list widget and list model, and then added some code to change the contents of the list model whenever the cursor direction changes.


## More bugs!

I only started working on the rebus bug, because I was working on the word suggestions bug. And I only started working on that bug because I discovered it while using the Editor. And it's a similar story with the word lists unification task. I only started working on that because I noticed the sidebar jumping bug.

After having fixed these two bugs, the plan was for me to focus on a bigger contribution---adding a step of lookahead to our fill algorithm. But alas, as I was fixing these two bugs, I noticed a few more bugs---but they shouldn't take too long, and they ought to be fixed. So I'm going to do that, and then transition to working on the fill lookahead bug.