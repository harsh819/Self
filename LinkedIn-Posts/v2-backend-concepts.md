# v2 — backend & API concept posts (replaces the old 12-post plan)

All posts are general engineering topics. None describe your employer's or a client's systems, so nothing here touches the confidentiality clause. Keep it that way when you edit: no "at my company", no client details.

Schedule: Tue / Thu / Sat, about 9–10 AM IST. Hashtags: 3 max.

| Week | Tue | Thu | Sat |
|---|---|---|---|
| 1 | `01-intro.md` (existing, one line updated) | Idempotency | Pagination |
| 2 | Status codes | Rate limiting | JWT vs sessions |
| 3 | Indexes | N+1 queries | Race conditions |
| 4 | Caching | Retries & backoff | `11-why-backend.md` (existing) |

Optional: end any post with a line from your own learning ("I tested this with a small NestJS + Postgres setup"), but only if you really did.

---

## Idempotency

Your API will receive the same request twice. Plan for it.

Networks time out. Clients retry. Users double-click. If POST /payments runs twice, someone is charged twice.

The fix is an idempotency key:
1. The client sends a unique key in a header (Idempotency-Key: <uuid>).
2. The server saves the key and the response the first time it processes the request.
3. A repeat request with the same key gets the saved response. Nothing runs again.

Details that matter:
→ Enforce it with a unique constraint in the database. "Check, then insert" can race.
→ Reject the same key sent with a different payload.
→ Expire old keys after a while.

GET, PUT and DELETE should already be idempotent. POST is the one you have to protect.

Which of your endpoints would cause damage if called twice?

#backend #api #softwareengineering

---

## Pagination

Offset pagination works until your table gets big.

LIMIT 20 OFFSET 100000 makes the database find 100,020 rows and throw away 100,000. Every page gets slower than the last. And if a row is inserted between page requests, the user sees a duplicate or misses one.

Cursor (keyset) pagination:
WHERE id < :last_seen_id ORDER BY id DESC LIMIT 20

→ Uses the index, so every page costs about the same
→ Stable when data changes underneath
→ Trade-off: you can't jump straight to page 50

My rule: offset for small admin tables, cursor for feeds and large lists. Sort on a unique column, or on (created_at, id) so ties don't break it.

Which one is your API using right now?

#backend #postgresql #api

---

## Status codes

Half the bugs in API integrations come from wrong status codes.

The ones people mix up:
→ 400: the request is malformed
→ 401: I don't know who you are (not authenticated)
→ 403: I know who you are, and you can't do this
→ 404: it doesn't exist
→ 409: it conflicts with the current state (duplicate, version mismatch)
→ 422: well-formed, but fails validation
→ 429: too many requests
→ 500: our bug. 503: temporarily unavailable

Common mistakes:
✗ Returning 200 with {"error": "..."} in the body. Clients and monitoring can't tell it failed.
✗ Returning 500 for bad user input. It pages the on-call for nothing.
✗ Using 403 where 404 is safer, because 403 confirms the resource exists.

Status codes are part of your API contract. Treat them that way.

Which one do you see misused most?

#api #backend #restapi

---

## Rate limiting

A simple way to rate limit an API: the token bucket.

→ A bucket holds up to N tokens
→ It refills at a steady rate (say 10 per second)
→ Each request takes one token
→ Empty bucket: return 429 with a Retry-After header

It allows short bursts but enforces an average rate. A fixed window counter can't: a client can send a full quota at the end of one window and another at the start of the next, which is double the limit in a moment.

Things people forget:
→ Counters kept in memory per server multiply your limit by the number of servers. Use a shared store like Redis.
→ Limit per user or API key, not only per IP. Many users share one IP behind a NAT.

How are you limiting traffic on your APIs?

#backend #redis #api

---

## JWT vs sessions

"Should I use JWT or sessions?" Here's how I think about it.

Sessions: the server stores the session, the client holds an ID.
✓ Easy to revoke, just delete it
✗ Needs a shared store when you have multiple servers

JWT: a signed token the server can verify without a lookup.
✓ Stateless, handy across multiple services
✗ Hard to revoke before it expires

The common mistake is a long-lived JWT with no way to revoke it. If it leaks, it works until it expires.

A safer pattern: a short-lived access token (around 15 minutes) plus a refresh token stored on the server, which you can revoke.

And remember: a JWT is signed, not encrypted. Anyone can read the payload. Never put secrets in it.

A single web app? Sessions are often simpler. Several services? JWT starts to pay off.

What do you default to?

#authentication #backend #security

---

## Indexes

A slow query is usually a missing index, not a slow database.

An index works like a book's index: instead of reading every page, the database jumps to the right one. Without it you get a sequential scan over every row.

How to check: put EXPLAIN ANALYZE in front of the query and look for "Seq Scan" on a big table.

Composite indexes have an order. An index on (a, b) helps filters on a, or on a and b. It does not help a filter on b alone (the leftmost prefix rule).

The costs:
→ Every index slows writes
→ Every index takes space
→ A boolean column rarely benefits

Add indexes for queries you actually run. Don't index everything "just in case".

What's the biggest speed-up an index ever gave you?

#postgresql #database #backend

---

## N+1 queries

The N+1 query problem looks fine in development and hurts in production.

You fetch 50 orders. That's 1 query.
Then for each order you fetch its customer. That's 50 more.
51 queries for one page.

With 10 rows in development you never notice. With real data, the page crawls.

Fixes:
→ A JOIN
→ One batched query: WHERE id IN (...)
→ Eager loading in your ORM
→ DataLoader if you use GraphQL

How to spot it: log your queries during a single request. If the same query repeats with a different ID, that's it.

Has an ORM ever hidden one of these from you?

#backend #orm #graphql

---

## Race conditions

Two requests read a balance of 100. Both subtract 80. Both write 20.

You just created 80 out of nothing. This is the lost update problem, and it comes from "read, check, write" done in application code.

Three ways to fix it:
1. Make the change atomic: UPDATE accounts SET balance = balance - 80 WHERE id = 1 AND balance >= 80
2. Lock the row inside a transaction: SELECT ... FOR UPDATE
3. Optimistic locking: add a version column and fail the write if the version changed

A transaction alone isn't enough. At Postgres's default isolation level (READ COMMITTED), two transactions can still read the same old value.

If your code does "read, then decide, then write", ask what happens when two requests run at once.

#postgresql #backend #concurrency

---

## Caching

Before you add a cache, answer three questions:
1. How stale is the data allowed to be?
2. How will it be invalidated?
3. What happens when it's empty?

The usual pattern is cache-aside: check the cache, on a miss read the database, then store the result with a TTL.

Pitfalls:
→ Stampede: a popular key expires and hundreds of requests hit the database at once. Add jitter to TTLs, or let one request rebuild it.
→ Caching per-user data under a shared key leaks it to other users.
→ Caching data that must be exactly right, like balances and stock counts.

A cache trades correctness for speed. Know which one you're giving up.

How do you decide what's worth caching?

#redis #caching #backend

---

## Retries and backoff

Retrying failed requests can fix an outage, or cause one.

Do it right:
→ Retry only transient failures: timeouts, 502, 503, 429. Never retry a 400.
→ Use exponential backoff: wait 1s, 2s, 4s, 8s, with a cap and a max attempt count.
→ Add jitter (a bit of randomness). Without it, every client retries at the same moment and hits the recovering server together.
→ Respect Retry-After when the server sends it.
→ Only retry requests that are safe to repeat, which is where idempotency keys come in.

A retry without backoff is a self-inflicted DDoS.

What does your retry policy look like?

#backend #reliability #distributedsystems
