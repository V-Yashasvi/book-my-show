# Concurrency Strategy

## Problem

500,000 users may attempt to reserve the same seat simultaneously.

Our system must guarantee:

- No double booking
- Low latency
- High throughput

---

## Strategy

We use a hybrid approach.

### Step 1

Acquire Redis lock.

```
SETNX seat_lock:A12
```

TTL = 30 seconds

If lock acquisition fails:

```
Seat Temporarily Unavailable
```

---

### Step 2

Read seat from PostgreSQL.

If

```
status == available
```

Update

```
status = held
held_by = userId
held_until = NOW() + 10 minutes
```

---

### Step 3

During booking confirmation

Use optimistic locking.

```
UPDATE seats
SET version = version + 1
WHERE version = oldVersion;
```

If zero rows updated

Booking failed because another transaction completed first.

---

## Why Redis instead of FOR UPDATE?

Redis

Pros

- Extremely fast
- Offloads DB
- Handles very high concurrency

Cons

- Extra infrastructure

FOR UPDATE

Pros

- Simple
- ACID

Cons

- DB bottleneck
- Connection pool exhaustion

---

## Final Decision

Redis SETNX for locking.

PostgreSQL for final consistency.