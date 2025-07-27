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

Q: How do you ensure high throughput in a Java-based microservice handling 1000+ TPS?

A:

- Use asynchronous I/O (e.g., Spring WebFlux or Netty).
- Connection pooling (e.g., HikariCP).
- Implement caching using Redis or Caffeine.
- Minimize GC pauses using G1/ZGC and proper memory tuning.
- Apply load balancing and circuit breakers (e.g., Resilience4j).

Q: Describe a time when you debugged a bottleneck in a distributed system.

A:
I once worked on a system with increasing latency in message consumption. After profiling, we found thread starvation due to blocking I/O. I refactored the consumer using a non-blocking reactive approach, which reduced latency by 45% and improved throughput by 60%.

