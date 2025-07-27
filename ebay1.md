ebay1
=====

Q: How would you design a distributed system to detect fraudulent transactions in real-time?

A:

- Ingestion: Use Kafka or Kinesis for ingesting transaction streams.
- Processing: Implement real-time fraud scoring using Apache Flink or Spark Streaming.
- Storage: Use Cassandra or DynamoDB for fast writes and reads; S3 or Hadoop for historical data.
- Rules Engine: Pluggable rule-based engine with ML hooks.
- Alerting: Publish high-risk events to an SQS/SNS or Kafka topic for downstream services (e.g., notification, enforcement).
- Security: Encrypt data at rest and in transit. Apply RBAC and auditing.

