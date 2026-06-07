coroot etcd
===========

  No, etcd is not currently supported in either the  coroot-cluster-agent  or the main  coroot  codebase (there are no references to  etcd  at all).

  However, yes, we can absolutely add it! In fact, adding  etcd  is even easier than Redis or Postgres because etcd natively exports Prometheus metrics
  over HTTP (usually at  http://<etcd-ip>:2379/metrics  or  2381/metrics ). No external exporter process is needed.

  Here is how we can implement it:
  ──────
  ### Step 1: Scrape etcd in  coroot-cluster-agent

  We would add a new target type  TargetTypeEtcd = "etcd" . Since etcd exposes standard Prometheus metrics natively over HTTP, the agent can scrape it
  using a standard HTTP collector.

  1. Add Target Type: In  metrics/target.go , add:
    TargetTypeEtcd TargetType = "etcd"

  2. Add Pod Auto-discovery: Scan for pods with etcd annotations (e.g.  coroot.com/etcd-scrape: "true" ) in  TargetFromPod :
    if pod.Annotations["coroot.com/etcd-scrape"] == "true" {
        t = &Target{
            Type: TargetTypeEtcd,
            Addr: net.JoinHostPort(pod.IP, cmp.Or(pod.Annotations["coroot.com/etcd-scrape-port"], "2379")),
        }
    }

  3. Implement Collector: Setup an HTTP client that scrapes the endpoint and registers it with the registry.
  ──────
  ### Step 2: Define Queries & Model in  coroot

  On the Coroot main application side, we would configure the queries to load the most critical etcd metrics:

  In  constructor/queries.go :

    qDB("etcd_up", `etcd_up`),
    qDB("etcd_server_has_leader", `etcd_server_has_leader`),
    qDB("etcd_db_size_bytes", `etcd_debugging_mvcc_db_total_size_in_bytes`),
    qDB("etcd_disk_wal_write_latency", `histogram_quantile(0.99, rate(etcd_disk_wal_write_duration_seconds_bucket[$RANGE]))`),
    qDB("etcd_disk_backend_commit_latency", `histogram_quantile(0.99, rate(etcd_disk_backend_commit_duration_seconds_bucket[$RANGE]))`),
    qDB("etcd_proposals_failed_total", `rate(etcd_server_proposals_failed_total[$RANGE])`),
  ──────
  ### Step 3: Add  auditor/etcd.go  for the UI

  We would create a new auditor file  auditor/etcd.go  to construct the dashboard reports and checks:

  1. Availability Check: Alert if  etcd_up  is  0  or if  etcd_server_has_leader  is  0  (which means the cluster has lost quorum).
  2. Database Size Chart: Monitor database total size in bytes ( etcd_db_size_bytes ) to watch out for database space exhaustion (etcd defaults to a
  2GB/8GB quota limit).
  3. Disk Write Latency Chart: Show WAL write and backend commit latencies, which are crucial because etcd is highly sensitive to slow disk I/O.
  4. Failed Proposals Chart: Chart the rate of failed/timed out consensus proposals.
  ──────
  ### Step 4: Register in Application Types

  We would register  ApplicationTypeEtcd  in  model/application_types.go  so that Coroot automatically categorizes etcd pods and routes them to the new
  ETCD inspection report section.

  Would you like me to start implementing this etcd integration in the codebase?
  
  
  
  
  
 
~~~~