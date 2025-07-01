+++
title      = 'Bugs, bugs, and more bugs!'
date       = '2025-06-30'
slug       = 'bugs-bugs-bugs'
categories = ['GSoC']
draft      = true
+++

In the past two weeks, I worked on two things:
* Squashing a rebus bug.
* Unifying the two suggested words lists into one.

## The rebus bug

A rebus cell is a cell that contains more than one letter. These aren't too common in crossword puzzles, but they do appear ocassionally---especially in harder puzzles.

The bug was that our word suggestions lists were not working for slots with rebus cells in them. More specifically, if your cursor was on a cell that's within `letters in rebus - 1` cells to the right of a rebus cell, then an assertion would fail, and the word suggestions list would be empty.

Anyway, the cause of this bug is that our intersection code (which is what generates the suggested words) was not accounting for rebuses at all! The intersection code calculates an offset value, which is the number of cells that the cursor's cell is offset by, from the first cell in the current slot.

But to get this value, all the code did was go through each cell in the slot, one by one, incrementing a counter along the way, until it found the cell that matches the cursor's cell. So regardless of if a slot is rebus-free, or if it's made entirely out of rebuses, the intersection function calculates the same offset.

