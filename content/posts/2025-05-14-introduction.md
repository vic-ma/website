+++
title      = 'Introducing my GSoC 2025 project'
date       = '2025-05-16'
categories = ['GSoC']
draft      = true
+++

I will be contributing to GNOME Crosswords, as part of the Google Summer of Code 2025 program. My project adds construction aids to the Crosswords editor. These aids provide hints, warnings, and statistics that help the user create better crossword puzzles.

## GNOME Crosswords

The [GNOME Crosswords](https://gitlab.gnome.org/jrb/crosswords) project consists of two parts:
* The [Crosswords player](https://flathub.org/apps/org.gnome.Crosswords), which you can use to play crossword puzzles.
* The [Crosswords editor](https://flathub.org/apps/org.gnome.Crosswords.Editor), which you can use to create crossword puzzles.

My project focuses on the Crosswords editor.

If you would like to learn more about GNOME Crosswords, check out the [GUADEC presentation](https://www.youtube.com/watch?v=fcQfpQLLzYo) that Jonathan Blandford, the creator of Crosswords, gave last year.

## Crossword construction

Creating a crossword is tricky. Creating a _good_ crossword is even trickier. There are many things to take into account. For example:
* Are the words interesting?
* Are there any words that are so uncommon as to feel unfair?
* Does the puzzle have a good variety of parts of speech?
* Is the grid rotationally symmetric?
* Are there any unchecked squares?

To learn more about the crossword construction process, check out [How to Make a Crossword Puzzle](https://www.nytimes.com/2018/09/14/crosswords/how-to-make-a-crossword-puzzle-the-series.html), from _The New York Times_, as well as [How to Create a Crossword Puzzle](https://www.youtube.com/watch?v=aAqQnXHd7qk), from _Wired_.

Crossword editing software can make the process easier---certainly not easy---but easier. For example, the GNOME Crosswords editor gives you a list of possible words for each row/column, taking into account any cells in the row/column that are already filled with a letter (a constraint on the list of possible words).

The goal
