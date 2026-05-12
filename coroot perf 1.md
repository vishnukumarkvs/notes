coroot perf 1
=============

FNV (Fowler-Noll-Vo) is a fast, non-cryptographic hash function. fnv.New64a() produces a 64-bit hash from sequential byte input using XOR + multiply.
Why it's needed: MetricHash is the sharding key in ClickHouse (Distributed(..., MetricHash)) and part of the sorting key (ORDER BY MetricName, MetricHash, ...). It must be deterministic — same metric name + labels must always produce the same hash — so related metrics land on the same shard and ClickHouse's merge-tree can order rows correctly.
Why replace LabelsToSignature? The old code was:
labels := make(map[string]string, ...)  // alloc map
labels[label.Name] = label.Value        // populate it
// ...
hash := promModel.LabelsToSignature(labels)  // compute hash from map
delete(labels, promModel.MetricNameLabel)    // cleanup
It built a map[string]string solely to compute a hash, then immediately discarded it. The new code avoids the map entirely — it iterates labels once into a sorted slice, then streams (metricName, key0, val0, key1, val1, ...) straight into an FNV hash. No unnecessary map allocation, no double hashing. FNV from stdlib (hash/fnv) is used because it's fast, deterministic, and zero-dependency.




columnBatch is a snapshot data carrier. It's a plain struct holding copies of the ClickHouse column pointers at a point in time.
Why it was introduced: The old save() method held b.lock during the entire ClickHouse INSERT query — a network round-trip. While one goroutine was waiting for the INSERT to complete, every concurrent Add() call was blocked.
The fix: snapshot() swaps the column pointers under the lock (microseconds) and returns a columnBatch pointing to the old data. Fresh empty columns replace them in MetricsBatch. Then saveBatch(batch) runs the INSERT outside the lock using the snapshot. Now Add() callers only wait for the column append, not the network write.
// Under lock — fast pointer swap
batch := b.snapshot()  // steals b.Timestamp, b.MetricHash, etc.
b.lock.Unlock()
// Outside lock — slow network INSERT
b.saveBatch(batch)
Same pattern used for the periodic ticker flush and the limit-based flush.



ack @def . From documentation, I see it supports HA with PG backend. So, will configure that and scale replicas.

On a sidenote,

I tried to check the code in `collector/metrics.go` .  I see whenever a request comes, we add it to a bucket, we have this batchLimit   = 10000. So, whenever this bucket fills, we do a clickhouse insert. 

And this is sequential, the lock is released when the insert is completed. So, any requests coming need to wait if ch insert is in progress, which might have resulted in timeout. Is this view correct?

If so, can we run ch insert in a background goroutine?