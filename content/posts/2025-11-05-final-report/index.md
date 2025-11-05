+++
title      = 'Google Summer of Code 2025 final report'
date       = '2000-11-06'
slug       = 'gsoc-9'
categories = ['GSoC']
draft      = true
+++

For Google Summer of Code 2025, I worked on [GNOME Crosswords](https://gitlab.gnome.org/jrb/crosswords). GNOME Crosswords consists of a crossword player called [Crosswords](https://flathub.org/en/apps/org.gnome.Crosswords) and a crossword editor called [Crossword Editor](https://flathub.org/en/apps/org.gnome.Crosswords.Editor). I worked on the crossword editor.


## Project summary

I improved GNOME Crossword Editor's [word suggestion algorithm](https://gitlab.gnome.org/jrb/crosswords/-/raw/master/data/images/edit-grid.png), by re-implementing it as a forward-checking algorithm. Previously, our word suggestion algorithm only considered the constraints imposed by the intersection where the cursor is. This resulted in frequent dead-end word suggestions, which led to user frustration.

To fix this problem, I re-implemented our word suggestion algorithm to consider the constraints imposed by every intersection in the current slot. This significantly reduces the number of dead-end word suggestions and leads to a better user experience. 

As part of this project, I also researched the field of constraint satisfaction problems and wrote a report on how we can use the AC-3 algorithm to further improve our word suggestion algorithm in the future.

I also performed a competitive analysis of other crossword editors on the market and wrote a detailed report, to help identify missing features and guide future development.


## Word suggestion algorithm improvements











<!-- ## Constraint satisfaction problem research -->


<!-- ## Competitive analysis -->