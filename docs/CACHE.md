# Cache Strategy

## Event Details

Key

```
event:{id}
```

TTL

```
3600 seconds
```

Invalidate

Whenever event details change.

---

## Seat Availability Count

Key

```
availability:{eventId}:{category}
```

TTL

```
30 seconds
```

Invalidate

Whenever seat status changes.

---

## Seat Map

Key

```
seatmap:{eventId}
```

TTL

```
24 hours
```

Invalidate

Only if seating layout changes.

---

## Never Cache

Individual seat availability.

Always read from:

- Redis lock
- PostgreSQL

Reason

Avoid stale booking information.

---

## Strategy

TTL

+

Targeted cache invalidation.