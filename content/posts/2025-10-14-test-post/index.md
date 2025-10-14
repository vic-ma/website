+++
title      = 'This is a test post'
date       = '2025-10-14'
slug       = 'gsoc-8'
categories = ['GSoC']
draft      = true
+++

In the past few weeks, I've been improving some test code that I had written.

## Refactoring time!

The first order of business was to refactor the tests. There was a lot of boilerplate code, which created visual clutter and made it tedious to add new test cases.

For example, take a look at this test:
```c
static void
test_egg_ipuz (void)
{
  g_autoptr (WordList) word_list = NULL;
  IpuzGrid *grid;
  g_autofree IpuzClue *clue = NULL;
  g_autoptr (WordArray) clue_matches = NULL;

  word_list = get_broda_word_list ();
  grid = create_grid (EGG_IPUZ_FILE_PATH);
  clue = get_clue (grid, IPUZ_CLUE_DIRECTION_ACROSS, 2);
  clue_matches = word_list_find_clue_matches (word_list, clue, grid);

  g_assert_cmpint (word_array_len (clue_matches), ==, 3);
  g_assert_cmpstr (word_list_get_indexed_word (word_list,
                                word_array_index (clue_matches, 0)),
                   ==,
                   "EGGS");
  g_assert_cmpstr (
    word_list_get_indexed_word (word_list,
                                word_array_index (clue_matches, 1)),
    ==,
    "EGGO");
  g_assert_cmpstr (
    word_list_get_indexed_word (word_list,
                                word_array_index (clue_matches, 2)),
    ==,
    "EGGY");
}
```
That's an awful lot of code just to say:
1. Use the `EGG_IPUZ_FILE_PATH` file.
1. Find clue matches for the 2-Across clue.
1. Assert that the results are `["EGGS", "EGGO", "EGGY"]`.

I had a few more test cases like it, and I wanted to add way more. So, I knew that I had to refactor everything.

### Use a `Fixture`