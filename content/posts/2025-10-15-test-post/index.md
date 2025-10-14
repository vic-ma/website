+++
title      = 'This is a test post'
date       = '2025-10-15'
slug       = 'gsoc-8'
categories = ['GSoC']
draft      = true
+++

In the past few weeks, I've been working on improving some test code that I had written.

## Refactoring time!

The first order of business was to refactor the tests. There was a lot of boilerplate code, which created visual clutter and made it difficult to add new test cases.

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

### What I did

I first did two things to remove most of the boilerplate code:
1. Add a test fixture that extracts the test setup code.
1. Add an helper function that extracts the assertion code.

This is what the test case looked like afterward:
```c
static void
test_egg_ipuz (Fixture *fixture, gconstpointer user_data)
{
  test_clue_matches (fixture->word_list,
                     fixture->grid,
                     IPUZ_CLUE_DIRECTION_ACROSS,
                     2,
                     (const gchar*[]){"EGGS", "EGGO", "EGGY", NULL});
}
```

This was a lot better, but I knew that I could take it even further with macro functions.

I made one for the test case definitions:
```c
define ASSERT_CLUE_MATCHES(DIRECTION, INDEX, ...)           \
  test_clue_matches (fixture->word_list,                    \
                     fixture->grid,                         \
                     DIRECTION,                             \
                     INDEX,                                 \
                     (const gchar*[]){__VA_ARGS__, NULL})
```

Which turned the `test_egg_ipuz` test case into this:
```c
static void
test_egg_ipuz (Fixture *fixture, gconstpointer user_data)
{
  ASSERT_CLUE_MATCHES (IPUZ_CLUE_DIRECTION_ACROSS, 2, "EGGS", "EGGO", "EGGY");
}
```

I also made a macro function for the test case declarations:
```c
#define ADD_IPUZ_TEST(test_name, file_name)     \
  g_test_add ("/clue_matches/" #test_name,      \
              Fixture,                          \
              "tests/clue-matches/" #file_name, \
              fixture_set_up,                   \
              test_name,                        \
              fixture_tear_down)
```

Which turned this:
```c
g_test_add ("/clue_matches/test_egg_ipuz",
            Fixture,
            EGG_IPUZ,
            fixture_set_up,
            test_egg_ipuz,
            fixture_tear_down);
```

Into this:
```c
ADD_IPUZ_TEST (test_egg_ipuz, egg.ipuz);
```

## An unfortunate bug

Now, picture this: you just finished refactoring your test code. You fix some style issues, do a final test run, check over of your diff one last time...and everything looks good, so you open and MR for it.

And then the unthinkable happens---the CI pipeline fails! And apparently, it's due to a test failure? But you ran your tests locally, and everything went fine. You run them again, just to double check, and yup, they still pass.

So...what then? A sporadic CI test failure? Well, let's just try re-running the pipeline and see what happens.

...

Nope. Still fails.

...

Rats.

And what's more, it's only the Flatpak job's test run that fails. The native job's test run works fine. What could possibly be the cause of this?

I'll spare you details, but in the end, I found out that the bug came from me accidentally freeing an object before it was done being used.

So, this was the fix:
```diff
@@ -94,7 +94,7 @@ test_clue_matches (WordList *word_list,
                    guint clue_index,
                    const gchar *expected_words[])
 {
-  g_autofree IpuzClue *clue = NULL;
+  const IpuzClue *clue = NULL;
   g_autoptr (WordArray) clue_matches = NULL;
   g_autoptr (WordArray) expected_word_array = NULL;
```