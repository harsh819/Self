I added Redis caching in front of PostgreSQL. Here's what I learned.

The reason: [FILL IN: what was slow or repeatedly queried].

What I cached: [FILL IN: e.g. read-heavy data that rarely changes].
What I didn't: data that has to be correct every time.

Result: [FILL IN: what improved. Only use real numbers, or say "noticeably faster" if you didn't measure].

What I'd tell my past self:
1. Decide how stale data is allowed to be before you cache it.
2. Decide how it gets invalidated. This is the hard part.
3. Measure before and after, or you're guessing.

A cache is a trade-off between speed and correctness. Know which one you're giving up.

How do you decide what's worth caching?

#redis #postgresql #backend
