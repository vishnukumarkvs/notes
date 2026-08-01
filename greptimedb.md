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

Recommendation for your case:
1. Index all three (cluster_name, site, app_name) with INVERTED INDEX — cheap, effective at low cardinality.
2. Put all three in PRIMARY KEY, ordered by which one is most consistently present in your filters (probably cluster_name first if it's the coarsest/most-always-filtered dimension).
3. Partition on just one column — whichever gives you a partition count roughly matching your number of datanodes (or a small multiple), and reasonably even data/write distribution. Don't partition on all three; let the indexes handle fine-grained pruning within each region instead.



Yes, both are reversible — with an important nuance on how the change actually propagates.

Indexes: yes, can be added or removed via ALTER TABLE

ALTER TABLE my_metrics MODIFY COLUMN site UNSET INVERTED INDEX;
ALTER TABLE my_metrics MODIFY COLUMN site SET INVERTED INDEX;


Same pattern works for FULLTEXT INDEX and SKIPPING INDEX.


Partitions: yes, real repartitioning DDL exists

This one surprised me — it's a proper, user-invoked SQL feature, not just a "recreate the table" workaround:

ALTER TABLE my_metrics SPLIT PARTITION ... ;
ALTER TABLE my_metrics MERGE PARTITION ... ;
ALTER TABLE my_metrics REPARTITION ON COLUMNS (...) (...);

Run manually — there's no automatic background repartitioning triggered by load or size. It's backed by a dedicated RepartitionProcedure in meta-srv, so it's a proper distributed operation (not instantaneous — it's coordinated across regions/datanodes, similar in spirit to a region migration).



Good news: yes, it will fully use the index — and this actually applies more broadly than you might expect.

Why it works

GreptimeDB's inverted index isn't a simple "exact value → rows" hash lookup — it's an FST (finite-state transducer) over all indexed values, and regex predicates get compiled into a DFA (deterministic finite automaton) that walks the FST directly. This is the same technique Loki and Prometheus's own TSDB use for regex label matching. Concretely:


Types of indexes


The three index types in GreptimeDB

Each targets a different query shape. Here's all three, side by side.

1. Inverted index — exact/regex match on discrete values

cluster_name STRING INVERTED INDEX

Value → row-position mapping via an FST, walked directly for equality or an IN(...) list, or via a DFA for regex (as we covered with app_name=~"akar|eywa"). Precise — no false positives.

Best for: tag/label columns with low-to-medium cardinality — cluster_name, site, status, env. This is what you'd use for structured PromQL-style label matchers.

2. Skipping index (bloom filter) — cheap probabilistic existence check

trace_id STRING SKIPPING INDEX

Per-row-group bloom filter: "could this row group possibly contain value X?" No → skip it for free. Yes → still have to read and confirm (false positives possible, never false negatives). Cheaper to build/store than inverted index since it doesn't track exact row positions.

Best for: higher-cardinality columns where an exact FST would bloat — trace_id, request_id, session_id. Good when you mainly need "rule out irrelevant row groups cheaply" rather than pinpoint lookup.

3. Fulltext index — for unstructured text, not tags

log_message STRING FULLTEXT INDEX WITH (analyzer='English', backend='bloom')

This is the one I hadn't detailed yet. Purpose-built for free-text columns like log messages — not for tag columns (the engine actually refuses to build it on tag/primary-key columns). It tokenizes text and indexes terms, powering two query functions:
- matches(col, "+error -timeout") — boolean free-text search (AND/NOT/phrase syntax)
- matches_term(col, 'connection refused') — exact term/phrase match



Use inverted index. Thousands of distinct pod names is comfortably within its sweet spot — not even close to where you'd need to consider bloom filter instead. Here's the reasoning, grounded in how each index actually scales:

The real dividing line: what does cost scale with?

- Inverted index (FST): storage cost scales with number of distinct values, and the FST compresses shared prefixes/suffixes between similar strings into shared structure — it doesn't store each value independently.
- Skipping index (bloom filter): storage cost scales with total rows / granularity (default: one bloom filter per 10,240 rows), completely independent of how many distinct values exist. It never gets "cheaper" just because cardinality is low, and it doesn't get structurally worse as cardinality rises either — but it trades exactness for cost, giving you only a coarse "maybe" that still requires reading the row group to confirm.

Why this favors inverted index specifically for pod_name

Kubernetes pod names are a near-ideal case for FST compression — they're not random strings, they're structured: <deployment>-<replicaset-hash>-<random-suffix>. Thousands of pods across a handful of deployments means huge shared-prefix overlap (myapp-7d9f8c6b5-xk2pl, myapp-7d9f8c6b5-9fq2m, ...), which is exactly what an FST is built to exploit. "Thousands of distinct values" isn't high cardinality in the sense that would concern an FST — that concern kicks in more around hundreds of thousands to millions of distinct values (think per-row UUIDs like trace_id, where even a compressed structure starts costing real space).

Bloom filter (skipping index) is the right call when either:
- cardinality is genuinely enormous (millions+), or
- you don't actually need exact positional lookup, just "rule out row groups cheaply"

Neither applies here — you want exact filtering on pod_name, and thousands of values is trivial for the FST.

Recommendation

pod_name STRING INVERTED INDEX

Skip "no index" too — with thousands of distinct pods, an unindexed filter on pod_name falls back to Parquet min/max + full row-group reads, which is exactly the slow-query pattern you started this conversation asking about.



no — existing primary key columns can't be reordered or removed after creation. But there's important nuance for your auto-created-via-remote_write case.

but its ok, ots almost automatic


What FST actually is

FST = Finite State Transducer. It's a classic automata-theory data structure — a generalization of a finite state machine where each transition can both consume input and emit output. In the indexing context (Lucene, GreptimeDB, etc.), it's used as an ordered map: string key → integer value, encoded as a compact directed acyclic graph (DAG) instead of a flat list or hash table.




