# IT Operations Homelab

Local Docker lab for a small IT operations stack:

| Service | URL | Role |
| --- | --- | --- |
| osTicket + MariaDB | `http://127.0.0.1:8080` | Help desk |
| LLDAP | `http://127.0.0.1:17170` | Identity directory (LDAP `3890`) |
| Uptime Kuma | `http://127.0.0.1:3001` | Monitoring + status page |

Everything binds to localhost. Secrets stay in gitignored `.env` files. Compose YAML and `.env.example` are what get committed.

**Walkthrough:** [Projects/docs/README.md](Projects/docs/README.md)

## What this repo covers

1. **Compose setup** — one folder per service, named volumes, healthchecks, no secrets in Git.
2. **osTicket first-boot failure** — missing `.env` left MariaDB with a blank password; osTicket sat in a restart loop until the DB volume was recreated.
3. **Ticket queue** — password lockout, VPN from home, finance share access, printer offline, then a help-desk outage.
4. **LLDAP** — lab users under `dc=homelab,dc=internal`; password reset on the user object, not in the ticket thread.
5. **Uptime Kuma** — monitors osTicket, LLDAP, and an Azure portfolio site. osTicket was stopped on purpose; Kuma went DOWN; the container was started again; the ticket was closed with detection / diagnosis / resolution.

## Layout

```text
Projects/
  osticket/          compose + .env.example
  lldap/
  uptime-kuma/
  docs/              write-up
  evidence/screenshots/
    osTicket/
    LLDAP/
    KumaMonitor/
  .github/workflows/  compose validation
```

## Run it

```bash
cd Projects/osticket && cp .env.example .env   # fill values
docker compose --env-file .env up -d

cd ../lldap && cp .env.example .env
docker compose --env-file .env up -d

cd ../uptime-kuma
docker compose up -d
```

CI copies the example env files and runs `docker compose config` for each stack on push/PR.

## Limits

- Laptop lab, not a production IdP. osTicket and LLDAP share personas by name; they are not LDAP-bound.
- Ticket mailboxes used `@homelab.com`; LLDAP uses `@homelab.internal`.
- Images are version-tagged, not digest-pinned. No TLS.
