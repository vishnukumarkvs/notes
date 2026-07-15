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
  
  


