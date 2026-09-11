coroot LMA
==========

## Kafka

1. New metrics/kafka/ native collector (modeled on metrics/postgres/collector.go):
- Dep: add github.com/twmb/franz-go (+ pkg/kadm for Admin API, preferred over sarama — maintained, pure-Go, KRaft compatible).
- Background snapshot() loop like pg-collector, Collect() exposes:
- kafka_up, kafka_scrape_error
- kafka_broker_info{broker_id,host,port,rack,version}
- kafka_topic_partitions{topic}, kafka_partition_leader{topic,partition}, kafka_partition_replicas/isr{topic,partition}, kafka_under_replicated_partitions, kafka_offline_partitions{topic,partition}
- kafka_topic_log_end_offset{topic,partition}, kafka_consumer_group_offset/lag{group,topic,partition}
- kafka_controller_id
- Auth: SASL PLAIN/SCRAM-SHA-256/512 (user/pass from Credentials/CredentialsSecret), TLS via existing common.TLSCredentials + DatabaseTLSConfig pattern. Broker list from target Addr (bootstrap host:9092).
- Keep JMX passthrough as-is: document Strimzi JMX-exporter sidecar + kafka-exporter scraped via existing coroot.com/scrape-metrics/metrics-port/metrics-path annotations. Native collector fills the gap JMX does poorly (topic inventory, ISR/URP, group lag). Prefix all new metrics kafka_ to not collide.
2. Wire into metrics/target.go + metrics/metrics.go:
- TargetTypeKafka="kafka", StartExporter case kafka.New(...), TargetFromPod: coroot.com/kafka-scrape=true, ...-port default 9092, ...-credentials-*, ...-tls-secret-*, ...-param-sasl-mechanism/security-protocol.
- TargetFromConfig: allow type: kafka, host, port via existing ApplicationInstrumentation.
- Reuse secret/TLS resolution loop in startExporters() — no change needed beyond new type.
3. Strimzi auto-discovery (small):
- Option A (recommended, no API change): in TargetFromPod, also auto-create Kafka target when pod has strimzi.io/cluster + strimzi.io/kind in (Kafka, KafkaPool) labels, unless explicitly opted out (coroot.com/kafka-scrape=false).
- Option B: metrics/ksm/ksm.go + generators.go: add customresourcestate generators for Kafka/KafkaTopic/KafkaUser CRDs (kube_kafka_*) — only if you want operator-level inventory.
4. Flags/config/docs:
- flags/flags.go: e.g. --track-kafka-consumer-lag (default true), --kafka-exclude-topics (__consumer_offsets default), --kafka-max-topics/groups.
- Update k8s/ RBAC examples + README snippet for Strimzi annotations.
5. Verify:
- Unit tests like metrics/postgres/*_test.go with fake Admin API; go vet/test ./metrics/kafka/...; manual e2e vs local Strimzi quickstart (KRaft, 3 brokers, 1 lagging consumer) checking new series in /metrics and remote-write.

