# IT Operations Homelab

I built a local IT operations lab with Docker Compose. osTicket handles incidents and a small knowledge base, LLDAP gives me a directory to practise users, groups and password changes, and Uptime Kuma monitors the local services plus my Azure portfolio. I created realistic help desk tickets, documented the resolution process, then deliberately stopped osTicket and used monitoring plus Docker logs to detect and recover the outage. I kept everything local and reproducible so the project costs essentially nothing to run.

Local help desk lab: **osTicket + MariaDB** on Docker, used as a real queue — not a screenshot of a login page.


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
