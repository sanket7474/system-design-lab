# Proximity Service - Requirements

## Overview

A Proximity Service enables users to search for nearby points of interest (POIs) such as restaurants, gas stations, ATMs, stores, hospitals, and other businesses within a specified geographic radius.

---

# Functional Requirements

## Nearby Search

1. Users should be able to search for nearby places using:
    - Search radius
    - Optional category filter

    Response:

    - Place ID
    - Name
    - Category
    - Distance
    - Coordinates
    - Rating

2. Support configurable search radii:

    - 500m
    - 1km
    - 5km
    - 10km
    - 50km

3. Results should only include POIs within the requested radius.
4. Users can filter by category:
5. Support incremental loading.
6. Results should be sorted by:

    1. Distance
    2. Relevance
    3. Popularity (optional)
    4. Rating (optional)
7. Support incremental loading.

8. Internal systems should be able to:

    - Create POI
    - Update POI
    - Delete POI
    - Update coordinates

    Updates are relatively infrequent compared to read traffic can be eventual consistent.
---

# Non-Functional Requirements

Support:
- 100M+ POIs globally
- Millions of searches per second
- Worldwide geographic coverage

---
 Availability

- 99.99% availability
- No single point of failure
- Multi-region deployment

---

Accuracy
Results must:

- Stay within requested radius
- Return exact distance calculations
- Avoid missing nearby POIs at cell boundaries

---

# Capacity Estimation

## Assumptions

- 100M POIs
- 100M DAU
- Average 5 searches/day

---

## Read QPS

```text
100M × 5 / 86400 ≈ 5,800 QPS average
```

Assuming 20x peak traffic:

```text
≈ 120,000 QPS peak
```

---

## Storage

Assume:

```text
100M POIs
500 bytes per POI
```

```text
100M × 500

≈ 50 GB
```

Even with indexing overhead:

```text
≈ 100-150 GB
```