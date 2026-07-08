# BookMyShow Database Schema

## Goals

- Prevent double booking
- Support seat holds
- Support payment retries
- Scale to 500K concurrent users

---

## users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## events

```sql
CREATE TABLE events (
    id UUID PRIMARY KEY,
    title VARCHAR(255),
    venue VARCHAR(255),
    event_date TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

## seats

```sql
CREATE TABLE seats (
    id UUID PRIMARY KEY,
    event_id UUID REFERENCES events(id),
    seat_number VARCHAR(20),
    category VARCHAR(30),
    status VARCHAR(20),
    held_by UUID,
    held_until TIMESTAMP,
    version INT DEFAULT 0
);
```

Status values

- available
- held
- booked

---

## bookings

```sql
CREATE TABLE bookings (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    event_id UUID REFERENCES events(id),
    amount NUMERIC(10,2),
    status VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);
```

Booking Status

- pending
- confirmed
- failed
- refunded

---

## Indexes

```sql
CREATE INDEX idx_seat_event
ON seats(event_id);

CREATE INDEX idx_booking_pending
ON bookings(status)
WHERE status IN ('pending','failed');
```

---

## Why UUID?

- Prevents ID guessing
- Safer APIs
- Supports distributed ID generation

---

## Why version column?

Supports optimistic locking during booking confirmation.