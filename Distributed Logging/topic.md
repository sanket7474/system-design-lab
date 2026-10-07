## Distributed Logging System

Design a highly scalable distributed logging system for **200+ microservices** that can collect, process, store, search, analyze, and archive application and infrastructure logs while ensuring reliability, low ingestion latency, security, trace correlation, and cost-efficient retention.

### 1. Functional Requirements
  - Collect logs from 200+ microservices
  - Support virtual machines, containers, databases, gateways, and legacy applications
  - Prefer JSON or OpenTelemetry-compatible records
- Log correlation
  - Correlate using `trace_id`, `span_id`, `request_id`, and `correlation_id`
  - Link logs across multiple microservices for the same transaction
- Log levels
  - TRACE
  - DEBUG
  - INFO
  - WARN
  - ERROR
  - FATAL
- Log enrichment
  - Add service name and version
  - Add region, cluster, namespace, pod, host, and environment
  - Add team ownership and deployment metadata
- Processing
  - Parse structured and approved unstructured formats
  - Normalize timestamps and severity
  - Handle multiline exception stack traces
  - Mask or redact sensitive information
  - Generate fingerprints for repeated errors
- Search and filtering
  - Search by time range, service, environment, severity, region, host, and event type
  - Search by trace ID, request ID, correlation ID, and error code
  - Support full-text and structured-field searches
- Live log viewing
  - Near-real-time log tailing
  - Filter live logs by service, instance, severity, and environment
- Alerting
  - Error-rate threshold alerts
  - New exception fingerprint alerts
  - Missing-log alerts
  - Abnormal volume increase or decrease alerts
  - Security and audit alerts
  - Pipeline health alerts
- Retention management
  - Hot, warm, cold, and archive tiers
  - Different retention policies by log category
  - Automatic movement and deletion using lifecycle policies
- Archive and restoration
  - Archive old logs to object storage
  - Restore archived logs for investigation or compliance

### 2. Non-Functional Requirements

**Scalability**
- Support 200+ microservices and approximately 2,000 running service instances
- Support 12,000 average log events/sec
- Support 50,000 sustained log events/sec
- Support temporary bursts up to 100,000 log events/sec
- Scale horizontally across collectors, brokers, processors, and storage nodes
- Support future growth to 400-500 microservices without architectural redesign

**Reliability**
- At-least-once delivery for error, security, and audit logs
- Best-effort delivery for debug logs
- Durable buffering during downstream failures
- Prevent acknowledged critical logs from being silently lost
- Support replay after processor or storage recovery
- Use stable event identifiers where deduplication is required

**Fault Tolerance**
- Retry transient failures using exponential backoff and jitter
- Route malformed events to a dead-letter stream
- Drop low-priority logs before error, security, or audit logs during severe capacity exhaustion

**Observability**
- Metrics: ingestion rate, throughput, latency, rejection rate, dropped events, and processing failures
- Monitor broker consumer lag, under-replicated partitions, and disk usage
- Monitor search latency, indexing failures, query rejections, and storage growth


### 3. Capacity Estimation

**Assumptions**
- Total microservices: 200
- Average instances per microservice: 10
- Total application instances: 2,000
- Average logs per instance per second: 5
- Infrastructure and platform overhead: 20%
- Peak traffic multiplier: 3x
- Average raw log size: 1 KB
- Hot retention: 7 days
- Warm retention: additional 23 days
- Archive retention: additional 335 days
- Stream-buffer retention: 24 hours
- Stream replication factor: 3
- Searchable-storage replication: 2 copies

**Instance Calculation**

```text
Total application instances = 200 microservices x 10 instances
                            = 2,000 instances
```

**Traffic Calculation**

```text
Application event rate = 2,000 instances x 5 events/sec
                       = 10,000 events/sec

Including 20% infrastructure overhead:
Average event rate = 10,000 x 1.20
                   = 12,000 events/sec

Peak event rate = 12,000 x 3
                = 36,000 events/sec
```

**Provisioned Capacity**
- Average expected load: 12,000 events/sec
- Expected peak load: 36,000 events/sec
- Initial sustained capacity: 50,000 events/sec
- Temporary burst capacity: 100,000 events/sec

**Daily Event Volume**

```text
Events/day = 12,000 x 86,400
           = 1,036,800,000 events/day
           approximately 1.04 billion events/day
```

**Daily Storage Estimation**

```text
Raw storage/day = 1.0368 billion events x 1 KB
                approximately 1.04 TB/day

Raw storage/month = 1.04 TB x 30
                  approximately 31.2 TB/month

Raw storage/year = 1.04 TB x 365
                 approximately 379.6 TB/year
```

**Network Throughput**

```text
Expected peak payload = 36,000 events/sec x 1 KB
                      approximately 36 MB/sec

Burst payload = 100,000 events/sec x 1 KB
              approximately 100 MB/sec
```

- Provision at least 150 MB/sec aggregate ingestion capacity after including protocol, TLS, metadata, retry, and batching overhead
- Use at least 2-5 Gbps network capacity for major ingestion segments

**Hot Storage Estimation**

Assume searchable primary data consumes 70% of raw size after compression and indexing, with two stored copies.

```text
Hot storage = 1.04 TB/day x 7 days x 0.70 x 2
            approximately 10.2 TB

Including 30% operational headroom:
Hot capacity approximately 13.3 TB
```

**Warm Storage Estimation**

```text
Warm storage = 1.04 TB/day x 23 days x 0.70 x 2
             approximately 33.5 TB

Including 30% operational headroom:
Warm capacity approximately 43.6 TB
```

**Searchable Storage Estimation**

```text
Total searchable capacity = 13.3 TB hot + 43.6 TB warm
                          approximately 56.9 TB
```

- Recommended allocated searchable capacity: 65-75 TB
- Extra capacity supports disk watermarks, node recovery, rebalancing, and index growth

**Archive Storage Estimation**

Assume archive compression reduces stored data to 35% of raw size.

```text
Archive storage = 1.04 TB/day x 335 days x 0.35
                approximately 121.9 TB

Including 20% headroom:
Archive capacity approximately 146 TB
```

**Streaming Buffer Storage**

Assume 24-hour retention, 50% compression, and replication factor 3.

```text
Buffer storage = 1.04 TB x 0.50 x 3
               approximately 1.56 TB

Including 40% headroom:
Required buffer capacity approximately 2.2 TB
```

- Recommended broker storage: 3-5 TB usable capacity

**Partition Estimation**

Assume a target of 1,500 events/sec per partition at 50,000 sustained events/sec.

```text
Required partitions = 50,000 / 1,500
                    approximately 34 partitions
```

- Recommended initial active-ingestion partitions: 48-64
- Separate topics or streams may result in approximately 96 total logical partitions
- At replication factor 3, 96 partitions create 288 physical partition replicas

**Growth Estimation**

Assume 30% yearly growth.

```text
Current baseline = 1.04 TB/day
Year 1           = 1.35 TB/day
Year 2           = 1.76 TB/day
Year 3           = 2.28 TB/day
```

- Initial design target: 2 TB/day sustained ingestion
- Scale-out target: 4 TB/day without architectural redesign