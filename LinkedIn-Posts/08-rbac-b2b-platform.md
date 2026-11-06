In a B2B platform, "anyone can sign up" is a bug.

I built a food-products delivery platform with three portals: customer, admin and rider. It handles payments and role-based access.

The decision I'm most glad I made: new customer accounts don't work until a super-admin approves them.

Why:
→ Only verified businesses get in, so no random sign-ups.
→ Admins and riders each see only what their role needs.
→ Approval is one clear step in the flow, easy to audit.

Stack: Next.js, Python, MySQL, Docker.

If you build for businesses, ask who should be allowed in before asking how fast they can sign up.

How do you handle access control in your projects?

#rbac #nextjs #softwarearchitecture
