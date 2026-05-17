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



So with Redis + PostgreSQL:
- Chunks + query state → Redis (eliminates the cache storage need)
- App DB (projects, configs, etc.) → PostgreSQL (already supported via NewPostgres, also required for HA primary lock)
- instance.uuid — trivial to generate in-memory or store in PG
- cloud-pricing — could be embedded or fetched on startup
Result: PVC fully eliminated. No local persistent storage needed at all. The pods become completely stateless, and you can scale replicas arbitrarily with no duplicate ClickHouse queries and no cold-cache problem.
The only caveat: the cloud-pricing manager loads a JSON dump from disk (loadFromFile) and writes an updated copy on a timer (os.CreateTemp + os.Rename). That's < 10 lines to redirect to memory or an embedded fallback.


## Redis store

All done. Here's a summary of the changes on the redis-cache-backend branch:
Changes made (13 files, +144/-40 lines + 247 new)
Configuration
- config/config.go — Added RedisURL string to Cache config (YAML: cache.redis_url)
- main.go — Wires cfg.Cache.RedisURL into cache.Config.RedisURL
New file
- cache/redis_store.go — Full Redis cache backend with:
  - redisStore struct wrapping go-redis client
  - Chunk storage: writeChunk / readChunks — serialized via existing chunk.Write/ReadFrom into LZ4-compressed binary blobs, stored with TTL
  - Index: sorted set per (project, query_hash) for efficient time-range queries
  - State: hash per project, keyed by query_hash, storing JSON with {query, last_ts, last_error}
  - Step: simple key storing the scrape interval for GetStep() reads
Core cache logic
- cache/cache.go — NewCache branches: if RedisURL set, init Redis client; skip SQLite state, disk scan, GC, and compaction goroutines
- cache/updater.go — updaterWorker checks GetPrimaryLock() each iteration when Redis mode; writeChunk delegates to Redis
- cache/client.go — QueryRange, GetStep, GetTo, GetStatus all delegate to Redis when configured
- cache/db.go — saveState, loadStates, deleteState, getMinUpdateTime, getStatus delegate to Redis
- cache/gc.go / compaction.go — Return early if Redis mode
Helper
- cache/chunk/chunk.go — Added ReadFrom(io.Reader, ...) for reading chunk data from byte buffers (used by Redis); refactored Read(path, ...) to call ReadFrom
How to use
# coroot.yaml
cache:
  redis_url: "redis://user:password@host:6379/0"
With Redis + PostgreSQL:
- Only the leader pod runs ClickHouse queries, others serve reads from Redis
- No PVC needed
- No duplicate CH queries
- No cold-cache on new pods


# redis_store.go

chunkKey — coroot:chunk:{project}:{query_hash}:{from_ts}
Stores the actual time-series data — the LZ4-compressed binary blob from chunk.Write(). The {from_ts} in the key lets you address a specific 10-minute window directly. TTL is set to the cache GC TTL (default 30 days).
indexKey — coroot:idx:{project}:{query_hash}
A sorted set (ZSET) that maps {query_hash} → all available chunk timestamps for that query. The score is from_ts, so you can do range queries like "give me all chunks between time X and Y". This replaces the in-memory chunksOnDisk map and the filesystem directory listing. Without this, you'd have to SCAN all coroot:chunk:* keys to find chunks for a specific query.
stateKey — coroot:state:{project}
A hash replacing the SQLite prometheus_query_state table. Each hash field is {query_hash} and the value is a JSON blob of {query, last_ts, last_error}. One hash per project — ~200 fields (one per PromQL query). This is O(1) to read all states for a project with HGETALL, vs. a SQL query.
stepKey — coroot:step:{project}
A plain string key storing the scrape interval (step) for the project. Set by the leader updater, read by all pods for GetStep(). In the file-based approach, this was inferred from the chunk metadata on disk. With Redis, non-leader pods have no local files, so this key ensures they can still determine the step without scanning all chunks.






