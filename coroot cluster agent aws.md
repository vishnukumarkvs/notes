coroot cluster agent aws
========================

This repository uses the AWS SDK for Go (v1) (https://github.com/aws/aws-sdk-go) primarily for discovering and collecting metrics from Amazon RDS and Amazon ElastiCache instances.
The implementation is centered in the metrics/aws/ directory. Here is a breakdown of how the SDK is utilized:
1. Resource Discovery
The Discoverer in metrics/aws/aws.go uses the SDK to scan the environment for active resources:
*   RDS: Calls rds.DescribeDBInstances to list all databases and rds.ListTagsForResource to apply tag-based filtering.
*   ElastiCache: Calls elasticache.DescribeCacheClusters and elasticache.ListTagsForResource to identify cache nodes.
2. RDS Metrics & Metadata
The RDSCollector in metrics/aws/rds.go extracts detailed information from the SDK's resource objects:
*   Metadata: Collects instance status, engine version, instance type, and storage configuration from the rds.DBInstance struct.
*   Enhanced Monitoring (OS Metrics): If enabled, it uses cloudwatchlogs.GetLogEvents to retrieve JSON-encoded OS-level metrics (CPU, Memory, Disk I/O) from the RDSOSMetrics log group.
3. RDS Log Collection
The LogReader in metrics/aws/logs.go implements a streaming-like log fetcher:
*   File Discovery: Uses rds.DescribeDBLogFiles to find available log files for an instance.
*   Log Ingestion: Uses rds.DownloadDBLogFilePortion to incrementally download log content (like Postgres/Aurora logs) for parsing and error detection.
4. ElastiCache Metadata
The ECCollector in metrics/aws/elasticache.go uses the elasticache.CacheCluster and elasticache.CacheNode structs provided by the SDK to collect instance status, engine details, and endpoint information.
5. Infrastructure & Configuration
*   Authentication: Uses github.com/aws/aws-sdk-go/aws/credentials to support static Access Key/Secret Key configuration.
*   Session Management: Uses github.com/aws/aws-sdk-go/aws/session to manage connections and retries.
*   Utilities: Uses github.com/aws/aws-sdk-go/aws/arn for parsing resource identifiers and github.com/aws/aws-sdk-go/aws/awserr for structured error handling.


