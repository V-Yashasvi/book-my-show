# BookMyShow System Architecture

```text
                           Mobile / Browser
                                  │
                                  ▼
                        CloudFront CDN
      ┌────────────────────────────────────────────┐
      │ Cache Hit → Static Assets, Event Pages     │
      │ Cache Miss → Forward API Requests          │
      └────────────────────────────────────────────┘
                                  │
                                  ▼
                 Application Load Balancer (ALB)
              • SSL Termination
              • Health Checks every 10 sec
              • Rate Limit: 200 req/IP/min
                                  │
        ┌──────────────┬──────────────┬──────────────┐
        ▼              ▼              ▼
   Node.js API    Node.js API    Node.js API × N
      │               │               │
      │───────────────┼───────────────│
      │
      ├──────── READ ───────────────► Redis Cluster
      │                               • event:{id} (TTL 3600s)
      │                               • availability:{event}:{category} (TTL 30s)
      │                               • seat_lock:{seatId} (SETNX, TTL 30s)
      │                               ← CONCURRENCY.md
      │
      ├──────── WRITE ──────────────► PostgreSQL Primary
      │                               • Seat Updates
      │                               • Booking Inserts
      │                               • Payment Updates
      │                               ← SCHEMA.md
      │
      ├──────── READ ───────────────► PostgreSQL Read Replica 1
      │                               Event Details
      │                               User Booking History
      │
      ├──────── READ ───────────────► PostgreSQL Read Replica 2
      │                               Seat Availability
      │
      └──────── Publish ────────────► SQS Payment Queue
                                      bookingId
                                      userId
                                      amount
                                      paymentToken
                                      Visibility Timeout: 30 sec
                                      Retry: 3
                                      DLQ → payment-dlq
                                      ← QUEUE.md
                                              │
                                              ▼
                           ECS Payment Workers ×10
                           1. Read Message
                           2. Call Razorpay
                           3. Update PostgreSQL
                           4. Publish SNS
                           5. Delete SQS Message
                                              │
                                              ▼
                                  Amazon SNS
                             ┌────────┴─────────┐
                             ▼                  ▼
                        Amazon SES          SMS Service
                           Email           Booking SMS
```

## Design References

- Redis SETNX Lock ← CONCURRENCY.md
- Async Payment Queue ← QUEUE.md
- Read Replicas ← SCHEMA.md
- Cache TTL Strategy ← CACHE.md