# Distributed Locking System - Requirements

## Overview

The Distributed Locking System provides a mechanism for coordinating access to shared resources across multiple services, instances, and regions.

In a distributed environment, multiple services may attempt to perform the same operation simultaneously, leading to race conditions, duplicate processing, inconsistent data, or deadlocks.

The Distributed Locking System ensures that only one client can acquire a lock on a resource at a given time while providing high availability, fault tolerance, lease expiration, and automatic lock recovery.

Typical use cases include:

- Preventing duplicate payment processing.
- Ensuring only one scheduler executes a job.
- Leader election.
- Inventory reservation.
- Distributed transaction coordination.
- Cache rebuild coordination.
- Preventing concurrent report generation.

---

# Functional Requirements

## Core Features

### Lock Management

- Acquire distributed lock.
- Release distributed lock.
- Renew lock lease.
- Check lock ownership.
- Check lock status.
- Force unlock (admin operation). in cases of deadlock
- Support lock expiration. TTL for locks to recover deadlock

### Lock Types

- Exclusive Lock.
- Shared Lock.
- Reentrant Lock.
- Read Lock.
- Write Lock.
- Fair Lock.
- Lease-Based Lock.

### Lock Lifecycle

- Automatic lock expiration.
- Heartbeat-based lock renewal.
- Dead lock recovery.
- Lock ownership validation.
- Lock timeout handling.

### Concurrency Management

- Atomic lock acquisition.
- Atomic lock release.
- Atomic lock renewal.
- Prevent lock stealing.
- Prevent split-brain scenarios.

### High Availability

- Multi-node lock service.
- Automatic failover.
- Leader election.
- Data replication.
- Quorum-based consistency.

### Monitoring

- Active lock monitoring.
- Lock wait monitoring.
- Lock contention reporting.
- Failed acquisition tracking.
- Expired lock tracking.

### Security

- Authentication.
- Authorization.
- RBAC support.
- Audit logging.
- Secure lock ownership validation.

---

# Non-Functional Requirements

| Requirement | Target |
|------------|---------|
| Lock acquisition latency | < 20 ms |
| Lock release latency | < 10 ms |
| Lock renewal latency | < 10 ms |
| Availability | 99.99% |
| Consistency | Strong Consistency |
| Fault Recovery Time | < 10 sec |
| Maximum Concurrent Locks | 10 Million |
| Throughput | 100K lock operations/sec |
| Lock Expiration Accuracy | ±1 second |
| RPO | 0 |
| RTO | < 5 minutes |

---

# Capacity Estimation

## Assumptions

| Metric | Value |
|----------|---------|
| Services Using Locks | 500 |
| Application Instances | 10,000 |
| Concurrent Active Locks | 10 Million |
| Lock TTL | 30 Seconds |
| Lock Operations/sec | 100,000 |
| Average Lock Record Size | 200 Bytes |
| Renewal Interval | 10 Seconds |
| Regions | 3 |
| Replication Factor | 3 |

---

# Write Traffic Calculation

Assume:

```text
100,000 lock operations/sec
```

Operations include:

```text
Acquire
Release
Renew
Force Unlock
```

### Peak Traffic

```text
100,000 × 2

=
200,000 operations/sec
```

---

# Lock Renewal Traffic

Assume:

```text
10 Million active locks
```

Renewal interval:

```text
10 seconds
```

Renewals:

```text
10,000,000 / 10

=
1,000,000 renewals/sec
```

Large-scale deployments typically require batching and optimized renewal strategies.

---

# Read Traffic Calculation

Assume:

```text
50,000 lock status checks/sec
```

Peak:

```text
100,000 reads/sec
```

---

# Network Bandwidth

## Lock Request Payload

Example:

```json
{
  "lockKey": "payment-order-123",
  "ownerId": "instance-567",
  "ttl": 30000
}
```

Average payload:

```text
200 Bytes
```

---

## Incoming Traffic

```text
100,000 operations/sec
×
200 Bytes

=
20 MB/sec
```

### Daily Traffic

```text
20 MB/sec
×
86400

=
1.7 TB/day
```

---

# Storage Estimation

## Lock Record

```text
Lock Key          64 Bytes
Owner ID          32 Bytes
TTL               8 Bytes
Created Time      8 Bytes
Expiry Time       8 Bytes
Metadata          80 Bytes
--------------------------------
Total ≈ 200 Bytes
```

---

## Active Lock Storage

```text
10,000,000 locks
×
200 Bytes

=
2 GB
```

Including indexes:

```text
4-6 GB
```

Including replication factor 3:

```text
12-18 GB
```

---

# Final Numbers

| Component | Estimate |
|------------|-----------|
| Active Locks | 10 Million |
| Lock Operations/sec | 100K |
| Peak Operations/sec | 200K |
| Renewal Requests/sec | 1 Million |
| Read Requests/sec | 50K |
| Daily Traffic | 1.7 TB |
| Active Storage | 2 GB |
| Replicated Storage | 12-18 GB |
| Audit Storage/day | 864 GB |
| Availability | 99.99% |
| Recovery Time | < 10 sec |
| Regions | 3 |
| Cluster Size | 5+ Nodes |

---