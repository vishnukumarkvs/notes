greptimedb
==========

1. File-level pruning (skip whole SST files, no I/O) — using manifest time-range stats (src/mito2/src/read/pruner.rs).
2. Row-group-level pruning — Parquet min/max statistics per row group let it skip whole row groups that can't match (src/mito2/src/sst/parquet/reader.rs:1271-1308). This works well for =, !=, <, >, IN, etc. It does not work for regex, since regex has no min/max range representation.
3. Optional index-based pruning — if you've built an inverted index / bloom filter / fulltext index on that column, it can skip row groups (even for regex) more precisely (src/mito2/src/sst/index/inverted_index/...). This only kicks in if an index actually exists for that column — it's not automatic for every tag/label column.



# How it works

Not on the datanode as primary storage — object storage (S3/GCS/Azure, or local disk in standalone mode) is the source of truth. Datanodes are compute nodes that read/write to that shared object store; they only keep a local write-cache in front of it (disabled by default, but usually enabled in example configs, write_cache_size = "5GiB").


1. Region pruning (frontend) — zero I/O, based on partition rules in metadata.
2. File pruning (datanode, in-memory manifest) — zero I/O, based on each SST's time range / PK range already cached in memory.
3. Footer + row-group-stats fetch — a small S3 GET (~64KB) per surviving file, to read Parquet min/max stats and consult the inverted-index/bloom-filter sidecar files (also small GETs). This is I/O, but tiny compared to the actual data.
4. Row-group pruning — done in-memory using what was fetched in step 3. Row groups that can't match are dropped here.
5. Actual data fetch — only now does it pull the real column data (the expensive GETs) for row groups that survived every prior filter.

