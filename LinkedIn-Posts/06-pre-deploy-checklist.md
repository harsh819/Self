My checklist before I push anything to production:

[EDIT THIS LIST — keep only what you really check]
☐ Does it work locally with production-like data?
☐ What else does this change touch? (other modules, other clients)
☐ Are environment variables and configs the same on the server?
☐ Is there a way to roll back, and have I thought about how?
☐ Is the database change safe on existing data?
☐ Are logs in place to tell me if it fails?
☐ Is the server healthy right now? (disk, memory, running processes)
☐ Is this the right time to deploy? (not during peak usage)

Carelessness on production costs more than slowness does. Ten minutes of checking has saved me hours of recovery.

What would you add to this list?

#devops #deployment #backend
