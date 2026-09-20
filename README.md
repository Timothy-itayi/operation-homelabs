# IT Operations Homelab

Local help desk lab: **osTicket + MariaDB** on Docker, used as a real queue — not a screenshot of a login page.

The work completed so far is the part a hiring manager can actually probe:

1. Stand up the desk as Compose (secrets out of Git, database unpublished, app on `127.0.0.1:8080`).
2. Recover the first boot when MariaDB initialized with a blank password and osTicket entered a restart loop.
3. Run ticket scenarios: password lockout, VPN from home, finance share access, printer offline, and a VPN recurrence.
4. Deploy LLDAP as the identity store and practice a password reset on the user object — not in the ticket thread.

**Read the case study:** [Projects/docs/README.md](Projects/docs/README.md)

| Service | Status |
| --- | --- |
| osTicket / MariaDB | Deployed and used |
| LLDAP | Deployed; users created; password reset practiced |
| Uptime Kuma | Not started |

Lab mailboxes in LLDAP are `@homelab.internal`. Live `.env` files are gitignored.
