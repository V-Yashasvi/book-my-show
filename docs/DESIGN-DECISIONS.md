# Design Decisions

---

## Decision: Redis SETNX vs PostgreSQL FOR UPDATE

### Context

The system must support 500,000 concurrent users without double booking.

### Options Considered

1. PostgreSQL FOR UPDATE
   - Strong consistency
   - High database contention
   - Limited by connection pool

2. Redis SETNX (Chosen)
   - Sub-millisecond locking
   - Offloads locking from database
   - Supports very high concurrency

### Why Chosen

Redis handles lock acquisition much faster than database row locking while PostgreSQL remains the source of truth during booking confirmation.

### Tradeoffs Accepted

- Redis becomes another critical dependency.
- Lock TTL must be managed carefully.

### Revision Trigger

If multi-region consistency becomes mandatory.

---

## Decision: Cache Invalidation

### Context

Seat availability changes frequently.

### Options

1. TTL Only
2. TTL + Event Driven Invalidation (Chosen)

### Why Chosen

TTL reduces database load while event-driven invalidation keeps availability reasonably fresh.

### Tradeoffs

Small stale-read window.

### Revision Trigger

If stale seat counts negatively impact user experience.

---

## Decision: UUID vs SERIAL

### Context

Booking IDs should not be predictable.

### Options

1. SERIAL
2. UUID (Chosen)

### Why Chosen

UUIDs improve security and support distributed ID generation.

### Tradeoffs

Larger index size.

### Revision Trigger

Internal-only deployment with sequential IDs.

---

## Decision: SQS Visibility Timeout

### Context

Payment processing takes approximately 10–15 seconds.

### Options

15 sec

20 sec

30 sec (Chosen)

### Why Chosen

Allows workers to complete payment before messages become visible again.

### Tradeoffs

Longer retry delay if worker crashes.

### Revision Trigger

Payment latency changes significantly.