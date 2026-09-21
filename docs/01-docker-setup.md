# 01 — Docker and repository setup

One Git repo, one folder per service. Compose is the contract. `.env` stays off Git.

## Layout

- `Projects/osticket/` — help desk + MariaDB
- `Projects/lldap/` — identity
- `Projects/uptime-kuma/` — monitoring
- `Projects/docs/` — this write-up
- `Projects/evidence/screenshots/{osTicket,LLDAP,KumaMonitor}/`
- `.gitignore` covers `.env`, logs, `backup/`, `secrets/`
- `.env.example` is the public template

## Commands

```text
docker compose --env-file .env config >/dev/null
docker compose --env-file .env up -d
docker compose ps
docker volume ls
```

`--env-file` is explicit on purpose. Compose will load a local `.env` if you are in the right directory; CI and a wrong cwd will not.

## Checks

- Docker Desktop shows each Compose project.
- osTicket publishes `127.0.0.1:8080->80/tcp`. MariaDB has no host port.
- LLDAP publishes `127.0.0.1:17170` and `127.0.0.1:3890`.
- Kuma publishes `127.0.0.1:3001`.
- `.env` is untracked. `.env.example` is not.

![Docker Desktop with the osTicket project running](../evidence/screenshots/Docker_Stack_Running.png)

osTicket and MariaDB as one Compose project. MariaDB is not published to the host. osTicket is on `8080:80`.

## Notes

- `127.0.0.1:8080:80` keeps the desk off the LAN. Bare `8080:80` would listen on every interface.
- `docker compose config` prints interpolated secrets. Safer: check variable *names*, then `config >/dev/null`.

```text
grep -E '^OST_' .env | cut -d= -f1
docker compose --env-file .env config >/dev/null && echo "Compose configuration OK"
```
