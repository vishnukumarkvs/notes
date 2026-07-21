greptimedb
==========

1. File-level pruning (skip whole SST files, no I/O) — using manifest time-range stats (src/mito2/src/read/pruner.rs).
2. Row-group-level pruning — Parquet min/max statistics per row group let it skip whole row groups that can't match (src/mito2/src/sst/parquet/reader.rs:1271-1308). This works well for =, !=, <, >, IN, etc. It does not work for regex, since regex has no min/max range representation.
3. Optional index-based pruning — if you've built an inverted index / bloom filter / fulltext index on that column, it can skip row groups (even for regex) more precisely (src/mito2/src/sst/index/inverted_index/...). This only kicks in if an index actually exists for that column — it's not automatic for every tag/label column.



