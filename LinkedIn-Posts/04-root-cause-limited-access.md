How do you debug a system when you only have limited, read-only access to the database?

That was my situation on the after-midnight shows bug. Here's what I did:

1. Wrote down exactly what the display should show and what it actually showed.
2. Picked specific late-night shows and checked how they were stored in the data.
3. Compared that with the query the display used to fetch "today's shows".
4. Found where the two disagreed: how the query decided which date a late show belongs to.

I didn't change anything on the client's side. I only read, compared and recorded.

Limited access isn't a blocker. It makes you precise: one hypothesis, one check, one result, written down.

What's your approach when you can't touch the system you're debugging?

#debugging #sql #backend
