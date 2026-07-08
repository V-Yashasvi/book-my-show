# Post Roast Updates

---

## Update 1: Per-user Seat Hold Limit

### Triggered By

Panel Question 3

### What Changed

Added Redis key

holds:{userId}

Maximum concurrent holds = 8

### Why

Prevents one user from blocking hundreds of seats.

### Cost

One additional Redis GET.

### Still Doesn't Solve

Multiple fake accounts.

---

## Update 2: Circuit Breaker for SQS

### Triggered By

Panel Question 4

### What Changed

If SQS publish fails for more than 60 seconds:

- Switch payment processing to synchronous mode.
- Notify users of slower processing.

### Why

Prevents complete booking outage.

### Cost

Higher API latency.

### Still Doesn't Solve

Reduced throughput during prolonged outages.

---

## Update 3: Peak Sale Hold TTL

### Triggered By

Panel Question 3

### What Changed

Seat hold reduced from 10 minutes to 3 minutes during flash sales.

### Why

Releases abandoned seats sooner.

### Cost

Less payment time for users.

### Still Doesn't Solve

Users with slow payment methods.