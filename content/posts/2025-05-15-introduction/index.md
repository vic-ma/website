+++
title      = 'Introducing my GSoC 2025 project'
slug       = 'gsoc-1'
date       = '2025-05-16'
categories = ['GSoC']
+++


I will be contributing to GNOME Crosswords, as part of the Google Summer of Code 2025 program. My project adds construction aids to the Crosswords editor. These aids provide hints, warnings, and data that help the user create better crossword puzzles.

I have three mentors:
* Jonathan Blandford
* Federico Mena Quintero
* Tanmay Patil


## GNOME Crosswords

The [GNOME Crosswords](https://gitlab.gnome.org/jrb/crosswords) project consists of two applications:
* The [Crosswords player](https://flathub.org/apps/org.gnome.Crosswords), which lets you play crossword puzzles.
* The [Crosswords editor](https://flathub.org/apps/org.gnome.Crosswords.Editor), which lets you create crossword puzzles.

My project focuses on the Crosswords *editor*.

To learn more about GNOME Crosswords, check out this [GUADEC presentation](https://www.youtube.com/watch?v=fcQfpQLLzYo) that Jonathan Blandford, the creator of Crosswords, gave last year.


## Crossword construction

Constructing a crossword puzzle is tricky. Constructing a *good* crossword puzzle is even trickier. The main difficulty lies in finding the right words to fill the grid.

Initially, it's quite easy. The grid starts off completely empty, so the first few words that you add don't have any restrictions on them (apart from word length). But as you fill the grid with more and more words, it becomes increasingly difficult to find words to fill the remaining rows/columns. That's because a remaining row, for example, will have a few of its cells already filled in by the words in the intersecting columns. This restricts the list of possible words for that row. 

Something you can run into is that halfway through the construction process, you realize that one or more rows/columns cannot be filled at all, because no word meets the constraints imposed on it by the prefilled cells! In that case, you would need to backtrack and delete some of the intersecting words and try again. Of course, trying to replace a word could lead to other words on the grid needing to be replaced too---so you can get a domino effect of groups of rows/columns on the grid needing to be redone!

And that's just what you have to deal with to create a valid crossword puzzle. To create a *good* crossword puzzle, there are many more things to consider, some of which impose further restrictions on the words that you can use. For example:

* Are the words interesting?
* Are there any words that are so uncommon as to feel unfair?
* Does the puzzle have a good variety of parts of speech?
* Is the grid rotationally symmetric?
* Are there any unchecked cells?

To learn more about the crossword construction process, check out [How to Make a Crossword Puzzle](https://www.nytimes.com/2018/09/14/crosswords/how-to-make-a-crossword-puzzle-the-series.html) by *The New York Times*, as well as [How to Create a Crossword Puzzle](https://www.youtube.com/watch?v=aAqQnXHd7qk) by *Wired*.

Suffice it to say, creating a good crossword puzzle is difficult. Thankfully, crossword construction software can make this process easier---certainly not easy---but easier. For example, the GNOME Crosswords editor gives you a list of possible words for each row/column, taking into account any cells in the row/column that are already filled with a letter. We can consider this feature a "construction aid."

The goal of my GSoC project is to add additional construction aids to the Crosswords editor. These aids will help the user create better crossword puzzles.


## Construction aids

Here's a list of potential construction aids that this project can add:
* Warning for unches (unchecked cells).
* Warning for non-dictionary-words.
* Warning for words with low familiarity.
* Indicator for average familiarity of words.
* Warning for crosswordese (overused crossword words).
* Heat map for hard-to-fill cells.
* Parts-of-speech distribution graph.

Right now, we are in the community bonding period of the GSoC program (May 8 to June 1). During this period, I will work with my mentors to determine which construction aid or aids this project should add, what they should look like, and how they should be implemented. By the end of the month, I will have created some design docs laying all this out. That will make it much easier to hit the ground running, once the coding period starts, in June.


## Mini crossword

Here's a [mini crossword](https://drive.google.com/file/d/1IjSUo3j_GK_Lw-x5mhFfX3qRLDZN2TOf) that I made! You can try it out by using the [Crosswords player](https://flathub.org/apps/org.gnome.Crosswords).

![My mini crossword](https://victorma.ca/posts/gsoc-1/mini.png)