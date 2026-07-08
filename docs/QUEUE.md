# Async Payment Queue

## Flow

User

↓

POST /bookings

↓

API Server

↓

Hold seats

↓

Create pending booking

↓

Publish message to SQS

↓

Return

202 Accepted

↓

Payment Worker

↓

Payment Gateway

↓

Update Booking

↓

Update Seats

↓

Send Email

↓

Send SMS

---

## SQS Message

```json
{
  "bookingId": "...",
  "userId": "...",
  "amount": 500,
  "paymentToken": "..."
}
```

---

## Visibility Timeout

30 seconds.

Reason

Maximum payment latency is approximately 15 seconds.

---

## Retry

Maximum

3 retries.

---

## Dead Letter Queue

Messages that fail after 3 retries move to DLQ.

CloudWatch monitors DLQ.

---

## Why Async?

Without async

- API waits for payment
- DB connections remain occupied
- Poor scalability

With async

- API responds in ~50ms
- Payment workers scale independently