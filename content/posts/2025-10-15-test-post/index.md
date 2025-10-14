+++
title      = 'This is a test post'
date       = '2025-10-15'
slug       = 'gsoc-8'
categories = ['GSoC']
draft      = true
+++

Over the past few weeks, I've been working on improving some test code that I had written.

## Refactoring time!

My first order of business was to refactor the test code. There was a lot of boilerplate, which made it difficult to add new tests, and also created visual clutter.

For example, have a look at this test case:
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
1. Run the `word_list_find_clue_matches()` function on the 2-Across clue.
1. Assert that the results are `["EGGS", "EGGO", "EGGY"]`.

And this was repeated in every test case, and needed to be repeated in every new test case I added. So, I knew that I had to refactor my code.

### Fixtures and functions

My first step was to extract all of this setup code:
```c
g_autoptr (WordList) word_list = NULL;
IpuzGrid *grid;
g_autofree IpuzClue *clue = NULL;
g_autoptr (WordArray) clue_matches = NULL;

word_list = get_broda_word_list ();
grid = create_grid (EGG_IPUZ_FILE_PATH);
clue = get_clue (grid, IPUZ_CLUE_DIRECTION_ACROSS, 2);
clue_matches = word_list_find_clue_matches (word_list, clue, grid);
```

To do this, I used a fixture:
```c
typedef struct {
  WordList *word_list;
  IpuzGrid *grid;
} Fixture;

static void fixture_set_up (Fixture *fixture, gconstpointer user_data)
{
  const gchar *ipuz_file_path = (const gchar *) user_data;

  fixture->word_list = get_broda_word_list ();
  fixture->grid = create_grid (ipuz_file_path);
}

static void fixture_tear_down (Fixture *fixture, gconstpointer user_data)
{
  g_object_unref (fixture->word_list);
}
```

My next step was to extract all of this assertion code:
```c
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
```

To do this, I created a new function that runs `word_list_find_clue_matches()` and asserts that the result equals an `expected_words` parameter.
```c
static void
test_clue_matches (WordList *word_list,
                   IpuzGrid *grid,
                   IpuzClueDirection clue_direction,
                   guint clue_index,
                   const gchar *expected_words[])
{
  const IpuzClue *clue = NULL;
  g_autoptr (WordArray) clue_matches = NULL;
  g_autoptr (WordArray) expected_word_array = NULL;

  clue = get_clue (grid, clue_direction, clue_index);
  clue_matches = word_list_find_clue_matches (word_list, clue, grid);
  expected_word_array = str_array_to_word_array (expected_words, word_list);

  g_assert_true (word_array_equals (clue_matches, expected_word_array));
}
```

After all that, here's what my test case looked like:
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
Much better!

### Macro functions

But as great as that was, I knew that I could take it one step further with macro functions.

I created a macro function to simplify test case definitions:
```c
#define ASSERT_CLUE_MATCHES(DIRECTION, INDEX, ...)          \
  test_clue_matches (fixture->word_list,                    \
                     fixture->grid,                         \
                     DIRECTION,                             \
                     INDEX,                                 \
                     (const gchar*[]){__VA_ARGS__, NULL})
```

Now, `test_egg_ipuz()` looked like this:
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

Now, picture this: You've just finished refactoring your test code. You add some finishing touches, do a final test run, look over your diff one last time...and everything looks good. So, you open up an MR and wait for a review.

But then, the unthinkable happens---the CI pipeline fails! And apparently, it's due to a test failure? But you ran your tests locally, and everything went fine. So you run the tests locally once again, just to be sure...and yup, they still pass.

So...what then? A sporadic test failure? A cosmic bit flip? Well, let's just try running the CI pipeline again and see what happens. Maybe the problem will go away.

...

Nope. Still fails.

...

Rats.

And what's more, it's only the Flatpak job's test run that fails. The native job's test run works fine. What could possibly be the cause of this?

Well, I'll spare you the gory details that it took for me to get to the answer. But the cause of the bug was me accidentally freeing an object that I should never have freed.

This meant that the corresponding memory segment *could be*---but, crucially, *did not necessarily have to be*---filled with garbage data. And this is why only the Flatpak job's test run failed...well, at first, anyway. By changing around some of the test cases, I was able to get the native job's test run and local test runs to fail. And that is what eventually clued me in to the true nature of this bug.

So, after the better part of two weeks, this was the fix:
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