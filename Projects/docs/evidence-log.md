# Evidence log

Rule: every image has a sentence saying what it proves. If it does not prove something, it does not belong in the case study.

## Captured

| File | Task | What it proves |
| --- | --- | --- |
| `evidence/screenshots/MariaDB-Rejects-Auth.png` | Deploy | First `docker compose config` with unset `OST_*` variables interpolating as blank strings. |
| `evidence/screenshots/resolve-Auth-Issue.png` | Deploy | MariaDB healthy while osTicket is `Restarting` — the auth/init mismatch. |
| `evidence/screenshots/Container-healthy.png` | Deploy | `down -v`, volumes recreated from `.env`, then `ps` with localhost publish. |
| `evidence/screenshots/Docker_Stack_Running.png` | Docker / Deploy | Compose project running in Docker Desktop; install completed in logs. |
| `evidence/screenshots/lldap-users.png` | LLDAP | Directory populated: admin plus alice, ben, chloe, dev, emma under `@homelab.internal`. |
| `evidence/screenshots/lldap-ben-password-reset.png` | LLDAP | `ben.flinder` user object with mailbox and **Modify password**. Recapture tighter (no browser chrome) if this repo goes public. |

Those four files currently include terminal dumps and installer logs. If this repository is published, crop env values and the install-time admin password from the log pane. The lab will be destroyed; a GitHub copy will not.

## Still needed (ticket UI)

Drop these into `Projects/evidence/screenshots/` then link them from [03 — osTicket ticket scenarios](03-osticket-scenarios.md).

| Suggested filename | What must be visible |
| --- | --- |
| `queue-open.png` | Open queue: `#326898` VPN, `#948476` printer. |
| `queue-closed.png` | Closed queue: `#795910` password, `#983225` access, `#344321` VPN. Closed By = Admin. |
| `users-directory.png` | Alice Saunders, Ben Flinder, Jessie Lemons. Crop personal mailboxes. |
| `ticket-344321-resolved.png` | Claim, Network Support, internal cause/action/prevention, customer reply, Resolved. |
| `ticket-795910-password-resolved.png` | Emergency, Password / Account Access, reply with no password in the body. |
| `ticket-983225-access-resolved.png` | Finance share request, High, resolution reply. |
| `ticket-948476-printer-open.png` | PDF works / office printer does not; Open. |
| `ticket-326898-vpn-open.png` | Recurrence, High, unassigned, SLA due date. |
| `staff-login.png` | Agent login page on `127.0.0.1:8080` (no password manager popup). |
| `lldap-alice-password-reset.png` | Alice’s user object *after* reset, `password_modified_date` later than create — closes the loop on `#795910`. |
| `lldap-groups.png` | Groups exist (or explicitly do not). Needed before `#983225` can claim a Finance grant. |
