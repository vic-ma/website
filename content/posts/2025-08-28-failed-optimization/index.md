+++
title      = 'When is an optimization not optimal?'
date       = '2025-08-28'
slug       = 'gsoc-7'
categories = ['GSoC']
draft      = true
+++

In the past few weeks, I've been working on the MR for my lookahead algorithm. And now, [it's finally merged](https://gitlab.gnome.org/jrb/crosswords/-/merge_requests/273)! There's still some more work to be done---particularly around the testing code---but the core change is finished, and is live.


## Suboptimal code?

While working on my MR, I noticed that one of my functions, `word_set_remove_unique ()`, could be optimized. Here is the function:
```c
void
word_set_remove_unique (WordSet *word_set1, WordSet *word_set2)
{
  g_hash_table_foreach_steal (word_set1, word_not_in_set, word_set2);
}

/* Helper function */
static gboolean word_not_in_set (gpointer key,
                                 gpointer value,
                                 gpointer user_data)
{
  WordIndex *word_index = (WordIndex *) key;
  WordSet *word_set = (WordSet *) user_data;
  return !g_hash_table_contains (word_set, word_index);
}
```

`word_set_remove_unique ()` takes two sets and removes any elements in the first set that don't exist in the second set. So essentially, it performs a set intersection, in-place, on the first set.

[My lookahead function](https://gitlab.gnome.org/jrb/crosswords/-/blob/d80c5792235e348348c9438e19b9a6bcdc20966b/src/clue-matches.c#L169) calls `word_set_remove_unique ()` multiple times, in a loop. My lookahead function always passes in the same persisted word set---`clue_matches_set`---as the first set. The second set changes with each loop iteration. So essentially, my lookahead function uses `word_set_remove_unique ()` to gradually refine `clue_matches_set`.

Now, importantly, `clue_matches_set` is sometimes larger than the second word set---potentially several orders of magnitude larger. And because of that, I realized that there was an optimization I could make.


## An obvious optimization

See, [`g_hash_table_foreach_steal ()`](https://docs.gtk.org/glib/type_func.HashTable.foreach_steal.html) runs the given boolean function on each element of the given hash table, and it removes the element if the function returns `TRUE`. And my code always passes in `word_set1` as the hash table (with `word_set2` being passed in as `user_data`). This means that the boolean function is always run on the elements of `word_set1` (`clue_matches_set`).

But `word_set1` is sometimes smaller than `word_set2`. This means that `g_hash_table_foreach_steal ()` sometimes has to perform this sort of calculation:
```c
WordSet *word_set1;  /* Contains 1000 elements. */
WordSet *word_set2;  /* Contains 10   elements. */

for (word : word_set1)
  {
    if (!g_hash_table_contains (word_set2, word))
      g_hash_table_remove (word_set1, word);
  }
```

This is clearly inefficient. The point of `word_set_remove_unique ()` is to calculate the intersection of two sets. It's only an implementation detail that it does this by removing the elements from the first set.

The function could also work by removing the unique elements from the second set. And in the case where `word_set1` is larger than `word_set2`, that would make more sense. It could be the difference between calling `g_hash_table_contains ()` (and potentially `g_hash_table_remove ()`) 10 times and calling it 1000 times.

So, I thought, I can optimize `word_set_remove_unique ()` by reimplementing it like this:
1. Figure out which word set is larger.
2. Run `g_hash_table_foreach_steal ()` on the larger word set.
3. If the second set was the larger set, then swap the pointers.

```c
/* Returns whether or not the pointers were swapped. */
gboolean
word_set_remove_unique (WordSet **word_set1_pp, WordSet **word_set2_pp)
{
  WordSet *word_set1_p = *word_set1_pp;
  WordSet *word_set2_p = *word_set2_pp;
  if (g_hash_table_size (word_set1_p) <= g_hash_table_size (word_set2_p))
    {
      g_hash_table_foreach_steal (word_set1_p, word_not_in_set, word_set2_p);
      return FALSE;
    }
  else
    {
      g_hash_table_foreach_steal (word_set2_p, word_not_in_set, word_set1_p);
      *word_set1_pp = word_set2_p;
      *word_set2_pp = word_set1_p;
      return TRUE;
    }
}
```


## Not so obvious?

Well, I made the change and...well...it's not really optimal. See, we have some profiling code that measures how long each frame takes to render. The output looks like something like this:
```
update_all():
    Average time (13.621 ms)
    Longest time (45.903 ms)
    Total iterations (76)
    Total number of iterations longer than one frame (27)
    Total time spent in this function (1035.203f ms)

word_list_find_intersection():
    Average time (0.774 ms)
    Longest time (4.651 ms)
    Total iterations (388)
    Total time spent in this function (300.163f ms)
``` 

And when I compared the results between the optimized and unoptimized `word_set_remove_unique ()`, it turned out that the "optimized" version ran either slower or about the same.

This didn't make sense to me. I wasn't necessarily expecting some massive performance boost---but I certainly didn't expect the performance to be worse. The only additional overhead is the size check and potential pointer swapping, and some bit of memory management from the calling function (my lookahead function):
```c
swapped = word_set_remove_unique (&clue_matches_set,
                                  &intersection_word_set);
if (swapped)
  {
    word_array_unref (initial_word_array);
    initial_word_array = intersection_word_array;
  }
else
  {
    word_array_unref (intersection_word_array);
  }
```

Maybe I messed up the implementation somewhere. But in any case, that just goes to show the value of profiling!

## More testing needed

So...that was going to be the blog post. But in writing this and reimplementing the optimization---the original code was lost, because I never committed it---it looks like the optimization may be working after all. So maybe I did mess up the implemention last time.