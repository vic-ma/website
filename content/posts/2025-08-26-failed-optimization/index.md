+++
title      = 'When is an optimization not optimal?'
date       = '2025-08-26'
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

[My lookahead function](https://gitlab.gnome.org/jrb/crosswords/-/blob/d80c5792235e348348c9438e19b9a6bcdc20966b/src/clue-matches.c#L169) calls `word_set_remove_unique ()` multiple times, in a loop. It always passes in the same persisted word set---`clue_matches_set`---as the first set. The second set changes with each loop iteration. So essentially, my lookahead function uses `word_set_remove_unique` to gradually refine `clue_matches_set`.

Now, importantly, `clue_matches_set` is sometimes larger than the second word set---potentially several orders of magnitude larger. And because of that, I realized that there was an optimization I could make.


## An obvious optimization

See, [`g_hash_table_foreach_steal ()`](https://docs.gtk.org/glib/type_func.HashTable.foreach_steal.html) runs the boolean function on each element in the hash table, and it removes the element if the function returns `TRUE`. And my code always passes in `word_set1` as the hash table (with `word_set2` acting as `user_data`).

But `word_set1` is sometimes smaller than `word_set2`. This means that `g_hash_table_foreach_steal ()` can sometimes  perform this sort of calculation:
```c
WordSet *word_set1;  /* Contains 1000 elements. */
WordSet *word_set2;  /* Contains 10   elements. */

for (word : word_set1)
  {
    if (!g_hash_table_contains (word_set2, word))
      g_hash_table_remove (word_set1, word);
  }
```

Clearly, this is inefficient. The point of `word_set_remove_unique ()` is to perform a set intersection on two sets. It's only an implementation decision to do this by removing the elements from the first set.

`word_set_remove_unique ()` could also work by removing the unique elements from the second set. And in the case where `word_set1` is much larger than `word_set2`, it makes more sense to do that---It could be the difference between calling `g_hash_table_contains ()` (and potentially `g_hash_table_remove ()`) 10 times and calling it 1000 times.

So, I thought, there's a better implementation for this function:
1. Figure out which word set is larger.
2. Run `g_hash_table_foreach_steal ()` on the larger word set.
3. If the second set was the larger set, then swap the pointers.

Something like this:
```c
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

```
Targeting 16.67 ms for a frame

update_all():
	Average time (9.226 ms)
	Longest time (33.599 ms)
	Total iterations (88)
	Total number of iterations longer than one frame (9)
	Total time spent in this function (811.912f ms)

word_list_find_intersection():
	Average time (1.381 ms)
	Longest time (7.285 ms)
	Total iterations (686)
	Total time spent in this function (947.504f ms)
```