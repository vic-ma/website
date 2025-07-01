+++
title      = 'Coding begins!'
date       = '2025-06-02'
slug       = 'gsoc-2'
categories = ['GSoC']
+++


Today marks the end of the community bonding period, and the start of the coding period, of GSoC.

In the last two weeks, I've been looking into other crossword editors that are on the market, in order to see what features they have that we should implement.

I found the process enlightening. It was interesting to try so many different versions of the same product. The UIs differ quite dramatically, the same feature may be implemented differently in each editor, and some features only exist in a few or even a single editor.

Some editors of note:
* [Ingrid](https://ingrid.cx/) is the editor I liked the most. It's got a really nice UI, and it has all the important features, without any extraneous ones.
* [Exet](https://viresh-ratnakar.github.io/exet.html) has a lot of word transformation tools that don't exist in other editors. That makes it well-suited for making cryptics.
* [Crosserville](https://www.crosserville.com/) has a few useful tools that search previously published crossword puzzles. While the editor itself is nothing special, the search tools certainly are.
* [PuzzleMe](https://puzzleme.amuselabs.com/pmm/puzzle-create) is interesting, because the PuzzleMe player is used by several well-known publications, like The Atlantic, and The New Yorker, and the Los Angeles Times. But their editor is not very good at all. It does have some AI integration, though, which is unique.
* [Crossword Weaver](https://www.crosswordweaver.com/index.html) is not very useable, but it is very nostalgic! It's a Windows-XP-era app, and it's still being sold to this day, for $39.95!

I compiled all my observations into a [findings document](https://pad.gnome.org/s/aGYPwTen5). I used this document to create a list of [potential feature ideas](https://pad.gnome.org/s/dxgb8XiY4) for Crosswords. (Plus, I sprinkled in some other feature ideas that I already had in mind.) This document will also be useful in the future, whenever a comprehensive overview of crossword editors on the market is needed---like when deciding what feature to implement next, or how best to implement a specific feature.

Eventually, through a discussion with my mentor, we decided that I should start by tackling a [bug that I found](https://gitlab.gnome.org/jrb/crosswords/-/issues/269). This will help me get more familiar with the fill algorithm code, and it will inform my decisions going forward, in terms of what features I should work on.