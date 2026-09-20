# Case study: standing up a help desk and working the queue

This is the hiring-manager path through the work completed so far. It is not a Docker tutorial. The point is whether I can stand up a service, recover it when configuration and state disagree, and then run a realistic ticket workflow on top of it.

Read in this order:

1. [Docker and repository setup](01-docker-setup.md) — why the lab is structured like an ops environment, not a pile of `docker run` commands.
2. [osTicket deployment](02-osticket-deployment.md) — how the stack was built, how the first start failed, and what proved it was actually usable.
3. [osTicket ticket scenarios](03-osticket-scenarios.md) — fake employees, real help-desk behaviour: claim, diagnose, reply, transfer, close.
4. [LLDAP deploy and identity tasks](04-lldap-identity.md) — directory users, password reset on the account object, not in the ticket.

Supporting notes:

- [Evidence log](evidence-log.md) — every screenshot and what it proves.
- Compose: [`../osticket/compose.yaml`](../osticket/compose.yaml), [`../lldap/compose.yaml`](../lldap/compose.yaml)
- Secret templates: [`../osticket/.env.example`](../osticket/.env.example), [`../lldap/.env.example`](../lldap/.env.example)

## What is live

| Piece | Status |
| --- | --- |
| osTicket + MariaDB | Deployed on `127.0.0.1:8080`. First boot failed on blank secrets; recovered by recreating the DB volume. |
| Ticket workflow | Five end-user tickets plus the installer ticket. Three resolved, two still open. |
| LLDAP | Deployed on `127.0.0.1:17170` (UI) and `127.0.0.1:3890` (LDAP). Lab users created. Password reset practiced on `ben.flinder`. |
| Uptime Kuma | Directory only. Not started. |

## The 60-second version

I treated a laptop like a small IT operations environment: Compose as the contract, secrets out of Git, the help desk bound to localhost, and MariaDB not published to the host. The interesting failure was not “the container crashed.” MariaDB came up healthy with an empty password because the data volume initialized on first boot. osTicket then sat in a restart loop. The fix was not another restart. It was supplying `.env` and destroying the volume so the database user was created with the intended password.

After the desk was up I ran it like a queue: password lockout, VPN from home, access to a finance share, and a printer offline. The VPN ticket is the one I would walk through in an interview — claimed, internal cause/action/prevention note, transferred to Network Support, customer reply without dumping internals, then closed as Resolved.

Identity is a separate service. LLDAP holds the lab employees. A password ticket is closed in osTicket without a credential in the thread; the reset is done on the user object in the directory. The two systems are not bound yet — same personas, different mailbox suffixes — which is a limitation I would own, not hide.
