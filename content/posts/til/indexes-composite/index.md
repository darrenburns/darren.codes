+++
title = 'Multi-column indexes - order matters!'
date = 2025-01-05T08:00:00Z
author = "Darren Burns"
draft = true
tags = ["til", "sql", "databases"]
+++

I've been reading a bit about databases recently, and realised that my knowledge of [*indexes*](https://en.wikipedia.org/wiki/Database_index) was a bit lacking. I knew they "make queries faster for indexed columns", and that they generally use [B-trees (balanced search tree)](https://en.wikipedia.org/wiki/B-tree), but that's about it.

[Use the Index, Luke!](https://use-the-index-luke.com/) has taught me just how important column order is in multi-column indexes.

The key learning I had was that the first column in a multi-column index is always searchable via the index, and that indexed columns in a multi-column index should be ordered by the most frequently used accessed column first (to increase the likelihood of the index being usable for a query).

If you define a multi-column index on columns `A` and `B` like so:

```sql
CREATE UNIQUE INDEX idx_ab ON my_table (A, B);
```

It can be thought of as both an index on `A` *and* an index on `(A, B)`, due to the underlying B-tree structure.

This means that if you query the table using both `WHERE A = 1 AND B = 1` *or only* `WHERE A = 1`, the index will be used to quickly find the relevant rows. 

However, if you query the table using only `WHERE B = 1`, the index cannot be used as the primary sorting column is `A`, and the query will be slower.

For a far better explanation and dive into why this is the case, I thoroughly recommend reading [Use the Index, Luke!](https://use-the-index-luke.com/).

{{<callout>}}
"[The more you think about what the B in B-Tree means, the better you understand B-Trees!](https://vimeo.com/73481096)"
{{</callout>}}
