+++
title      = 'Google Summer of Code final report'
date       = '2025-11-07'
slug       = 'gsoc-9'
categories = ['GSoC']
draft      = true
+++

For Google Summer of Code 2025, I worked on [GNOME Crosswords](https://gitlab.gnome.org/jrb/crosswords). GNOME Crosswords is a project that consists of two apps: 
* [Crosswords](https://flathub.org/en/apps/org.gnome.Crosswords), a crossword player
* [Crossword Editor](https://flathub.org/en/apps/org.gnome.Crosswords.Editor), a crossword editor.

My work focused on the editor.

## Links

Here are links to everything that I worked.

### Merge requests

Merge requests related to the word suggestion algorithm:
1. [Improve word suggestion algorithm](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/273)
1. [Add `word-list-tests-utils.c`](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/286)
1. [Refactor `clue-matches-tests.c` by using a fixture](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/295)
1. [Use better test assert macros](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/296)
1. [Add macro to reduce boilerplate code in `clue-matches-tests.c`](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/307)
1. [Add a macro to simplify the `test_clue_matches` calls](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/310)
1. [Add more tests to `clue-matches-tests.c`](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/312)
1. [Use string parameter in macro function](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/313)
1. [Add performance tests to `clue-matches-tests.c`](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/314)
1. [Make phase 3 of `word_list_find_intersection()` optional](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/317)
1. [Improve print functions for `WordArray` and `WordSet`](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/320)

Other merge requests:
1. [Fix and refactor editor puzzle import](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/211)
1. [Add MIME sniffing to downloader](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/225)
1. [Add support for remaining divided cell types in `svg.c`](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/227)
1. [Fix intersect sort](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/249)
1. [Fix rebus intersection](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/251)
1. [Use a single suggested words list for Editor](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/256)

### Design documents

[Work in progress.](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/338)

### Other documents

Development:
* **[Ideas list](https://gitlab.gnome.org/jrb/crosswords/-/wikis/ideas)**
* **[Editor roadmap thoughts](https://gitlab.gnome.org/jrb/crosswords/-/wikis/editor-roadmap)**
* **[Crossword Editor architecture notes](https://gitlab.gnome.org/jrb/crosswords/-/wikis/guadec-notes)**
* **[Naming problems](https://gitlab.gnome.org/jrb/crosswords/-/wikis/naming-problems)**
* **[Font testing](https://gitlab.gnome.org/jrb/crosswords/-/wikis/font-testing)**

Word suggestion algorithm:
* **[CSP notes](https://gitlab.gnome.org/jrb/crosswords/-/wikis/csp-notes)**
* **[Miscellaneous papers](https://gitlab.gnome.org/jrb/crosswords/-/wikis/papers)**
* **[Sub-alphabet idea](https://gitlab.gnome.org/jrb/crosswords/-/wikis/sub-alphabet)**

Competitive analysis:
* **[Survey of existing crossword editors](https://gitlab.gnome.org/jrb/crosswords/-/wikis/survey-editors)**
* **[Survey of printing feature in existing editors](https://gitlab.gnome.org/jrb/crosswords/-/wikis/survey-printing)**

Other:
* **[Review of docs](https://gitlab.gnome.org/jrb/crosswords/-/wikis/docs-review)**

### Blog posts

1. [Introducing my GSoC 2025 project](https://victorma.ca/posts/gsoc-1/)
1. [Coding begins](https://victorma.ca/posts/gsoc-2/)
1. [A strange bug](https://victorma.ca/posts/gsoc-3/)
1. [Bugs, bugs, and more bugs!](https://victorma.ca/posts/gsoc-4/)
1. [My first design doc](https://victorma.ca/posts/gsoc-5/)
1. [It's alive!](http://victorma.ca/posts/gsoc-6/)
1. [When is an optimization not optimal?](http://victorma.ca/posts/gsoc-7/)
1. [This is a test post](http://victorma.ca/posts/gsoc-8/)

### Journal

I kept a [daily journal](https://pad.gnome.org/s/qszU26K2b) of the things that I was working on.

## Project summary

I improved GNOME Crossword Editor's word suggestion algorithm, by re-implementing it as a forward-checking algorithm. Previously, our word suggestion algorithm only considered the constraints imposed by the intersection where the cursor is. This resulted in frequent dead-end word suggestions, which led to user frustration.

To fix this problem, I re-implemented our word suggestion algorithm to consider the constraints imposed by every intersection in the current slot. This significantly reduces the number of dead-end word suggestions and leads to a better user experience.

As part of this project, I also researched the field of constraint satisfaction problems and wrote a report on how we can use the AC-3 algorithm to further improve our word suggestion algorithm in the future.

I also performed a competitive analysis of other crossword editors on the market and wrote a detailed report, to help identify missing features and guide future development.


## Word suggestion algorithm improvements

The goal of any crossword editor software is to make it as easy as possible to create a good crossword puzzle. To that end, all crossword editors provide a [*word suggestion list*](https://gitlab.gnome.org/jrb/crosswords/-/raw/master/data/images/edit-grid.png)---a list of words that fit the current slot. This feature helps the user find words that fit the slots on their grid.

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

Now, suppose that the current slot is 4-Across. The basic algorithm only considers the constraints imposed by the current slot, and so it returns all words that match the pattern `W O R _`---such as *WORD* and *WORM*. But none of these word suggestions actually fit in the slot---they all cause 4-Down to become some nonsensical word.

The problem is that the basic algorithm only looks at the current clue, 4-Across. It does not also look at other slots, like 4-Down. Because of that, the algorithm doesn't realize that 4-Down causes 4-Across to be unfillable. And so, the algorithm generates incorrect word suggestions.

### Our word suggestion algorithm

Our word suggestion algorithm was a bit more advanced than this basic algorithm. Our algorithm considered two constraints:
* The constraints imposed by the current slot.
* The constraints imposed by the intersecting slot where the cursor is.

This means that our algorithm could actually handle the problematic grid properly if the cursor is on the bottom-right cell. But not if the cursor is on any other cell of 4-Across:

![Broken behaviour](https://victorma.ca/posts/gsoc-6/broken.png)

### Consequences

All this means that our word suggestion algorithm was prone to generating *dead-end words*---words that seem to fit a slot, but that actually lead to an unfillable grid.

In the problematic grid example I gave, this unfillability is immediately obvious. The user fills 4-Across with a word like *WORM*, and they instantly see that this turns 4-Down into *ZERM*, a nonsense word. That makes this grid not so bad.

The worst cases are the insidious ones, where the fact that a word suggestion leads to an unfillable grid is not obvious at first. This leads to a ton of wasted time and frustration for the user.

### The fix

To fix this problem, I re-implemented our word suggestion algorithm to account for the constraints imposed by *all* the intersecting slots. Now, with the problematic grid example, our word suggestion algorithm correctly returns an empty list:

![Fixed behaviour](https://victorma.ca/posts/gsoc-6/fixed.png)

Our new algorithm doesn't eliminate dead-end words entirely. After all, the algorithm only checks the intersecting slots---it does not check the intersecting slots of the intersecting slots, and so on.

However, the constraints imposed by a slot onto the current slot become weaker, the further away it is. Consider: for a slot that's two intersections away from the current slot to constrain the current slot, it must first constrain the slot one intersection away enough for it to then constrain the current slot.

And so, although my changes do not eliminate dead-end words entirely, they do significantly reduce their prevalence, making for a much better user experience.

## Constraint satisfaction problems research




## Competitive analysis


<!-- ## Conclusion
other projects
Thank jonathan, creator...-->