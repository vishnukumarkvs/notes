coroot tracing
==============

Based on the code analysis of the  coroot  backend, I can answer your questions regarding how Coroot fetches traces, the timeouts you're experiencing, and the potential for scaling.

  ### 1. "By default it uses a 1-hour window and loads everything. Is this correct?"

  No, Coroot does not literally load "everything" (all spans) into memory or the UI.
  When you load the Traces UI page for a 1-hour window, the code in  api/views/tracing/tracing.go  executes two concurrent queries against ClickHouse:

  1. A Heatmap/Histogram Query: It aggregates all spans for the selected service in that 1-hour window to build the visual heatmap.
  2. A Sample Spans Query: It fetches a strict sample of only 100 spans ( LIMIT 100 ) for the selected criteria, rather than all traces.

  So, loading the aggregated heatmap data alongside exactly 100 sample spans is the intended and correct behavior.

  ### 2. "I see request timed out as well. So, is there any code inefficient in the current implementation?"

  Yes, there are two major inefficiencies in the ClickHouse schema and queries that cause timeouts under high trace volume.

  The timeout occurs because ClickHouse is taking too long to execute the trace queries. Here is why:

  • On-the-fly Heatmap Aggregation: In  clickhouse/traces.go  (inside  GetSpansByServiceNameHistogram ), Coroot queries the raw  otel_traces  table and performs a  GROUP BY  on  Timestamp  and  Duration  to generate the heatmap. Because there is no pre-aggregated Materialized View for
  histograms in the current schema, if your service has thousands of requests per second, ClickHouse has to scan and aggregate millions of raw span rows on the fly.
  • Parent Span Lookup by TraceId is Unoptimized: To calculate service dependencies and clients, Coroot fetches the parent spans of those 100 sample traces ( GetParentSpans ). The primary key of the  otel_traces  table is  ORDER BY (ServiceName, SpanName, toUnixTimestamp(Timestamp),
  TraceId) . Because a parent span can belong to a different service, the query cannot filter by  ServiceName  and must filter purely by  TraceId . This means ClickHouse cannot use the primary key effectively. Instead, it relies on a Bloom Filter skip index ( idx_trace_id ). If the 100
  sampled spans are spread across the full 1-hour window, ClickHouse is forced to scan a massive amount of data across all services for that hour using the bloom filter, which is very inefficient and a common bottleneck that leads to timeouts.

  ### 3. "Can scaling Clickhouse shards or Coroot replicas solve this?"

  • Scaling ClickHouse Shards: YES.
  Because the timeout is caused by heavy  GROUP BY  aggregations and massive data scans, adding more ClickHouse shards (using distributed tables) will significantly help. ClickHouse is designed to distribute these specific queries across shards, processing the aggregation in parallel and
  reducing the overall query time. Scaling up ClickHouse's vertical resources (more CPU/Memory, faster NVMe drives) will also directly solve this.
  • Scaling Coroot Replicas: NO.
  Coroot is a stateless API backend in this context; it just forwards the query to ClickHouse and waits. The bottleneck is strictly the ClickHouse database execution time. Scaling Coroot replicas will not prevent the queries from timing out.
  
  


 That is an excellent point. You are completely right—because ClickHouse Materialized Views act like triggers on new inserts, the new  otel_traces_histogram  table will start completely empty at the moment of migration.





------------------------------\






  If we simply updated the Go code to query the new table, your heatmap for the last 1 hour, day, or week (before the restart) would indeed appear blank.

  Fortunately, there are two standard ways to handle this so you do not lose your historical heatmap visibility:

  ### Solution 1: Backfilling the Data (Recommended)

  You can manually run a one-time script (or add it to your deployment steps) to backfill the historical data into the new table. Because we are just running the exact same  SELECT  query that the MV uses, we can take all the old traces and forcefully push them into the new table.

  You would execute this query directly in ClickHouse right after the migration:

    INSERT INTO otel_traces_histogram
    SELECT
        toStartOfInterval(Timestamp, INTERVAL 1 minute) AS Timestamp,
        ServiceName,
        (ParentSpanId = '') AS IsRootSpan,
        roundDown(Duration/1000000, [0, 5, 10, 25, 50, 100, 250, 500, 1000, 2500, 5000, 10000, inf]) AS DurationBucket,
        count() AS TotalCount,
        countIf(StatusCode = 'STATUS_CODE_ERROR') AS ErrorCount
    FROM otel_traces
    WHERE (SpanKind = 'SPAN_KIND_SERVER' OR SpanKind = 'SPAN_KIND_CONSUMER')
       OR ParentSpanId = ''
    GROUP BY Timestamp, ServiceName, IsRootSpan, DurationBucket;

  Note: This query will take some time to run because it does the "slow" math over all your historical data. But it only runs once! After it finishes, your new table will contain all the historical heatmap data, and the MV will handle all new data seamlessly.

  ### Solution 2: A Fallback in the Go Code

  If you don't want to run a heavy backfill query on your production database, we can update the Coroot Go code ( clickhouse/traces.go ) to perform a "fallback".

  We would write the Go code like this:

  1. Query the fast  otel_traces_histogram  table for the requested time window.
  2. If the query returns data, use it to draw the heatmap.
  3. If the query returns zero rows (meaning the user is looking at historical data from before the migration), the Go code automatically falls back and runs the old, slow query against the raw  otel_traces  table.

  This means old data will still be visible (though it might still be slow/timeout if you query 1 hour of heavy traffic), but new data will be lightning fast. Eventually, once your data retention period passes (e.g., 7 days), the old data is deleted anyway, and the fallback code is
  naturally never triggered again.

  Which path feels better for your environment? If you'd like, we can write the exact code patch to implement the Go-level fallback!
  
  
  
---------------------------------------------------



