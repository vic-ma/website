+++
title      = 'Google Summer of Code final report'
date       = '2025-11-07'
slug       = 'gsoc-9'
categories = ['GSoC']
draft      = true
+++

For Google Summer of Code 2025, I worked on [GNOME Crosswords](https://gitlab.gnome.org/jrb/crosswords). GNOME Crosswords consists of a crossword player called [Crosswords](https://flathub.org/en/apps/org.gnome.Crosswords) and a crossword editor called [Crossword Editor](https://flathub.org/en/apps/org.gnome.Crosswords.Editor). I worked on the crossword editor.


## Project summary

I improved GNOME Crossword Editor's word suggestion algorithm, by re-implementing it as a forward-checking algorithm. Previously, our word suggestion algorithm only considered the constraints imposed by the intersection where the cursor is. This resulted in frequent dead-end word suggestions, which led to user frustration.

To fix this problem, I re-implemented our word suggestion algorithm to consider the constraints imposed by every intersection in the current slot. This significantly reduces the number of dead-end word suggestions and leads to a better user experience.

As part of this project, I also researched the field of constraint satisfaction problems and wrote a report on how we can use the AC-3 algorithm to further improve our word suggestion algorithm in the future.

I also performed a competitive analysis of other crossword editors on the market and wrote a detailed report, to help identify missing features and guide future development.



## Word suggestion algorithm improvements

The goal of any crossword editor software is to make it as easy as possible to create a good crossword puzzle. To that end, all crossword editors provides a [*word suggestion list*](https://gitlab.gnome.org/jrb/crosswords/-/raw/master/data/images/edit-grid.png)---a dynamic list of words that fit the current slot. This feature helps the user find words that fit the slots on their grid.

In order to generate the word suggestion list, crossword editors use a *word suggestion algorithm*. The simplest example of a word suggestion algorithm considers two constraints:
* The size of the current slot.
* The letters in the current slot.

So for example, if the current slot is `C A _ S`, then this basic word suggestion algorithm would return all four-letter words that start with *CA* and end in *S*---such as *CATS* or *CABS*, but not *COTS*.

### The problem

There is a problem with this basic word suggestion algorithm, however. Consider the following grid:
```
+---+---+---+---+
|   |   |   | Z |
+---+---+---+---+
|   |   |   | E |
+---+---+---+---+
|   |   |   | R |
+---+---+---+---+
| W | O | R |   | < current slot
+---+---+---+---+
```
4-Down begins with *ZER*, so the only word it can be is *ZERO*. This constrains
the bottom-right cell to the letter *O*.

4-Across starts with *WOR*. We know that the bottom-right cell must be *O*, so
that means that 4-Across must be *WORO*. But *WORO* is not a word. So, 4-Down
and 4-Across are both unfillable, because no letter fits in the bottom-right
cell. This means that there are no valid word suggestions for either 4-Across or 4-Down.

Now, suppose that the current slot is 4-Across. The basic algorithm only considers the constraints imposed by the current slot, and so it returns all words that match the pattern `W O R _`---such as *WORD* and *WORM*. None of these words actually fit in 4-Across, however. We know that 4-Across is unfillable, so these words must all turn 4-Down into a nonsensical word (like *ZERD* or *ZERM*).

The problem is that the basic algorithm only looks at the current clue. It does not look at the intersecting clues of the current clue. Because of that, the algorithm is not able to see that 4-Down causes 4-Across to be unfillable. And so it generates incorrect word suggestions.


### Our word suggestion algorithm

Our word suggestion algorithm was a little bit more advanced than this basic alorithm---but not by much. So, our algorithm also could not handle the problematic grid properly:

![Broken behaviour](https://victorma.ca/posts/gsoc-6/broken.png)


### The fix

To fix this, I reimplemented our word suggestion algorithm as a forward-checking algorithm. Now, our algorithm considers the constraints imposed by the current slot as well as every intersecting slot.

![Fixed behaviour](https://victorma.ca/posts/gsoc-6/fixed.png)


## Constraint satisfaction problems research




## Competitive analysis
