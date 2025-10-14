+++
title      = 'This is a test post'
date       = '2025-10-15'
slug       = 'gsoc-8'
categories = ['GSoC']
draft      = true
+++

In the past few weeks, I've been working on improving some test that I had written.

## Refactoring time!

The first order of business was to refactor the tests. There was a lot of boilerplate code, which made it difficult to add new tests and created visual clutter.

For example, take a look at this test case:
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

So, before adding any new tests, I knew that I had to first refactor my code.

### Fixtures and functions

My first step was to extract all of this setup code, which was repeated at the start of each test case:
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

Next, I wanted to extract all of this assertion code:
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

To do this, I created a function that runs `word_list_find_clue_matches()` and asserts that the result equals an `expected_words` array that I pass in.
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

### Macro functions

That was a lot better, but I knew that I could take it one step further with macro functions.

I made a macro function for the test case definitions:
```c
#define ASSERT_CLUE_MATCHES(DIRECTION, INDEX, ...)          \
  test_clue_matches (fixture->word_list,                    \
                     fixture->grid,                         \
                     DIRECTION,                             \
                     INDEX,                                 \
                     (const gchar*[]){__VA_ARGS__, NULL})
```

Which turned the `test_egg_ipuz` test case definition into this:
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

Now, picture this: You've just finished refactoring your test code. You make some finishing touches, do a final test run, look over your diff one last time...and everything looks good, so you open and MR for it.

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

## More tests

Then, with that done, I went on to add some more tests. 