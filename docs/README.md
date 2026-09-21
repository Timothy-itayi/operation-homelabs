# Homelab write-up

How the stack was built, what broke, and how tickets were worked. Screenshots live under `Projects/evidence/screenshots/` in `osTicket/`, `LLDAP/`, and `KumaMonitor/`.

| Doc | What it covers |
| --- | --- |
| [01 — Docker setup](01-docker-setup.md) | Repo layout, secrets, localhost binds |
| [02 — osTicket deploy](02-osticket-deployment.md) | Compose design, blank-password first boot, recovery |
| [03 — Ticket scenarios](03-osticket-scenarios.md) | VPN, password, access, printer |
| [04 — LLDAP](04-lldap-identity.md) | Directory users, password reset |
| [05 — Uptime Kuma](05-uptime-kuma.md) | Monitors, status page, simulated outage |
| [Evidence log](evidence-log.md) | Image index |

Compose files: [osticket](../osticket/compose.yaml) · [lldap](../lldap/compose.yaml) · [uptime-kuma](../uptime-kuma/compose.yaml)
