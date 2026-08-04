# Nearby Friends System Design

## Overview

The system allows users to discover friends within a configurable radius (e.g., 5 km).

Each mobile client periodically sends its current location to the backend every **30 seconds**. Friends may:

- Stay within the radius
- Move into the radius
- Move out of the radius
- Stop sharing location

The system must continuously reflect these changes.

---

# Functional Requirements

## Core Features

1. User can enable/disable location sharing.
2. Client uploads location every 30 seconds.
3. User can view nearby friends.
4. Friends entering the radius should appear.
5. Friends leaving the radius should disappear.
6. Support configurable radius (1km, 5km, 10km, etc.).

---

# Non-Functional Requirements

| Requirement | Target |
|------------|----------|
| Location update frequency | 30 sec |
| Nearby query latency | < 200ms |
| Availability | 99.9% |
| Scalability | Millions of users |
| Real-time accuracy | Within 30-60 sec |

---

# Capacity Estimation

## Assumptions

| Metric | Value |
|----------|---------|
| Daily Active Users | 10 Million |
| Concurrent Active Users | 1 Million |
| Location Update Interval | 30 sec |
| Average Friends Per User | 200 |
| Nearby Search Radius | 5 km |
| Location Payload | 100 bytes |

---

# Write Traffic Calculation

Each active user sends one location update every 30 seconds.

```text
1,000,000 concurrent users
/
30 sec

= 33,333 updates/sec
```

### Peak Traffic (2x Buffer)

```text
≈ 70,000 writes/sec
```

---

# Read Traffic Calculation

Assume each active user opens the nearby screen once every 2 minutes.

```text
1,000,000 users
/
120 sec

= 8,333 reads/sec
```

### Peak Read Traffic

```text
≈ 20,000 reads/sec
```

---

# Network Bandwidth

## Incoming Traffic

Location update:

```json
{
  "userId":"123",
  "lat":18.52,
  "lon":73.85,
  "ts":123456789
}
```

Approx:

```text
100 bytes/update
```

Traffic:

```text
33,333 * 100

= 3.3 MB/sec
```

Daily:

```text
3.3 * 86400

≈ 285 GB/day
```

---

# Location Storage

Store only latest location.

### Location Record

```text
User Id      = 8 bytes
Latitude     = 8 bytes
Longitude    = 8 bytes
Timestamp    = 8 bytes
Metadata     = 30 bytes

Total ≈ 64 bytes
```

For 10 million users:

```text
10,000,000 * 64

≈ 640 MB
```

Including overhead:

```text
~ 2 GB
```

---

# Historical Location Storage

Assume:

```text
10M users
1 update / 30 sec
```

Updates per day:

```text
10,000,000
×
2 updates/min
×
60
×
24

=
28.8 Billion updates/day
```

Storage per update:

```text
50 bytes
```

Daily storage:

```text
28.8 B * 50

≈ 1.44 TB/day
```

# Nearby Query Estimation

Assume:

```text
5km radius
Urban city
```

Average result size:

```text
50 nearby friends
```

Response:

```text
50 * 100 bytes

≈ 5 KB
```

Read traffic:

```text
8,333 reads/sec
×
5 KB

≈ 41 MB/sec
```


# Final Numbers

| Component | Estimate |
|------------|------------|
| DAU | 10 Million |
| Concurrent Users | 1 Million |
| Location Writes | 33K/sec |
| Nearby Reads | 8K/sec |
| Peak Throughput | 70K/sec |
| Redis Storage | 5-8 GB |
| Kafka Traffic | 285 GB/day |
| Historical Data | 1.4 TB/day |
| WebSocket Connections | 1 Million |
| API Servers | 30 |
| WebSocket Servers | 15 |
| Nearby Response Size | 5 KB |
