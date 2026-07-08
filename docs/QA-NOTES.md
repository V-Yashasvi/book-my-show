# Panel Q&A Notes

---

## Question 1

What happens if Redis crashes?

Answer

- Redis Cluster provides high availability.
- If the cluster is unavailable, booking returns HTTP 503.
- PostgreSQL optimistic locking prevents duplicate booking.

Gap

Complete Redis outage still affects booking throughput.

---

## Question 2

Replica lag shows incorrect seat availability.

Answer

Read replicas are only used for browsing.

Booking always validates against PostgreSQL Primary.

Gap

Users may briefly see stale availability.

---

## Question 3

One user holds 200 seats.

Answer

Redis counter limits each user to 8 held seats.

Gap

Multiple fake accounts remain possible.

---

## Question 4

SQS becomes unavailable.

Answer

CloudWatch detects queue failure.

Circuit breaker switches to synchronous payment processing.

Gap

Lower throughput during outage.

---

## Question 5

Why Redis instead of FOR UPDATE?

Answer

Redis handles much higher concurrency.

PostgreSQL performs final booking confirmation.

Gap

Redis increases infrastructure complexity.