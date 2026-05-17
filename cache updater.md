cache updater
=============

Option 1: Shared RWX filesystem (no code changes)
- Extend PG advisory lock to the cache updater (1-line change)
- Use a ReadWriteMany PVC (NFS, EFS, Longhorn) instead of ReadWriteOnce
- Only the leader writes chunks; all pods read from the same shared volume
- No cache replication needed — all pods see the same files
- Downside: NFS latency on reads, potential file lock issues on write

Option 2: Redis (needs cache layer rewrite)
- Store chunks as binary blobs in Redis: coroot:cache:<project>:<query_hash>:<from_ts> → LZ4 compressed chunk bytes  
- Rewrite cache/client.go (QueryRange) and cache/updater.go (write) to use Redis instead of file I/O
- Pro: fast reads, built-in TTL eviction, no SPOF on storage
- Con: operational dependency, serialization overhead for every API request, memory sizing


(PG advisory lock). Extending GetPrimaryLock() check into updaterWorker is ~3 lines — only the primary runs the cache update loop; replicas serve reads only




The byProject in-memory index is rebuilt from Redis on startup instead of from disk scan.
Leader election for the updater: already partially done via db/lock.go (PG advisory lock). Extending GetPrimaryLock() check into updaterWorker is ~3 lines — only the primary runs the cache update loop; replicas serve reads only.
Compaction and GC need similar treatment — only the leader runs them, or they operate on Redis directly (safe since Redis is shared).
Read redundancy: all pods can read from the same Redis instance, so dashboards work identically regardless of which pod serves the request, and there's no cold-cache problem on new pods.
Worth it? Depends on your scale. For 2-3 replicas, the current local-cache approach is simpler and fast. If you're at 5+ replicas or ClickHouse query load is actually painful, this is a clean path forward.