# 02 — osTicket deployment

### Objective

Deploy osTicket with MariaDB so agents can log into a staff console and end users can open tickets — and prove the stack is actually usable, not merely “Up.”

### Why this matters

This is the same shape as a Cloud Support incident: the process is running, the dependency looks healthy, and the application still cannot do its job. If I cannot explain *why* a healthy database and a restarting app go together, I am guessing.

### What I implemented

Two services, one network, two volumes:

```1:45:Projects/osticket/compose.yaml
services:
  mariadb:
    image: mariadb:11
    container_name: itops-mariadb
    environment:
      MARIADB_RANDOM_ROOT_PASSWORD: "yes"
      MARIADB_DATABASE: osticket
      MARIADB_USER: osticket
      MARIADB_PASSWORD: ${OST_DB_PASSWORD}
    volumes:
      - osticket-db:/var/lib/mysql
    healthcheck:
      test: ["CMD", "healthcheck.sh", "--connect", "--innodb_initialized"]
      interval: 10s
      timeout: 5s
      retries: 10
    restart: unless-stopped

  osticket:
    image: rinkp/osticket-dockerized:1.18.4
    container_name: itops-osticket
    depends_on:
      mariadb:
        condition: service_healthy
    environment:
      OST_SECRET_SALT: ${OST_SECRET_SALT}
      OST_ADMIN_EMAIL: ${OST_ADMIN_EMAIL}
      OST_ADMIN_PASSWD: ${OST_ADMIN_PASSWD}
      OST_HELPDESK_URL: http://localhost:8080
      OST_HELPDESK_ONLINE: "true"
      OST_DBTYPE: mysql
      OST_DBHOST: mariadb
      OST_DBNAME: osticket
      OST_DBUSER: osticket
      OST_DBPASS: ${OST_DB_PASSWORD}
      OST_TABLE_PREFIX: ost_
    ports:
      - "127.0.0.1:8080:80"
    volumes:
      - osticket-attachments:/var/www/attachments
    restart: unless-stopped
```

Design choices that are not cargo-cult:

| Choice | Why |
| --- | --- |
| `depends_on` + `service_healthy` | osTicket must not race a database that has not finished InnoDB init. |
| MariaDB healthcheck `--innodb_initialized` | “mysqld is listening” is weaker than “storage is ready.” |
| `MARIADB_RANDOM_ROOT_PASSWORD` | The app uses the `osticket` user. A reusable root password on a laptop is extra attack surface. |
| Named volumes for DB and attachments | Tickets and files survive a container recreate. They also survive a *bad* first boot, which is the actual incident below. |
| Image `rinkp/osticket-dockerized:1.18.4` | Unattended install from env vars, so the lab is rebuildable. That is a supply-chain tradeoff: community image, version-tagged, not digest-pinned. |

### Procedure

First start was the wrong start: Compose with no `.env`.

```text
docker compose config          # WARN: OST_* unset, interpolated as blank strings
docker compose up -d           # images pull, network/volumes create, MariaDB goes healthy
docker compose ps              # osticket Restarting
```

Fix path:

```text
# create .env from .env.example (values stay local)
grep -E '^OST_' .env | cut -d= -f1
docker compose --env-file .env config >/dev/null
docker compose down
docker volume ls               # osticket-db still present — this is the trap
docker compose down -v         # destroy the volume that initialized with a blank password
docker compose --env-file .env up -d
docker compose ps
```

### Verification

1. `docker compose ps`: MariaDB `healthy`; osTicket bound to `127.0.0.1:8080`.
2. MariaDB logs: `ready for connections`.
3. osTicket logs: unattended install completed, helpdesk URL set to `http://localhost:8080`, attachment storage plugin enabled.
4. Browser: staff login at `/scp/login.php` returns the agent console.

### Evidence to capture

| File | What it must prove |
| --- | --- |
| `MariaDB-Rejects-Auth.png` | WARN lines for unset `OST_*` vars and rendered config with empty `MARIADB_PASSWORD` / `OST_DBPASS`. |
| `resolve-Auth-Issue.png` | After a naive up, MariaDB is healthy while osTicket is `Restarting`. |
| `Container-healthy.png` | `down -v`, volumes recreated, then `ps` shows healthy/starting on `127.0.0.1:8080`. |
| `Docker_Stack_Running.png` | Install finished in the UI logs. |

![Compose config with blank secrets](../evidence/screenshots/MariaDB-Rejects-Auth.png)

**What this proves:** the failure was not a bad image pull. Compose interpolated empty strings and still started the stack.

![MariaDB healthy, osTicket restarting](../evidence/screenshots/resolve-Auth-Issue.png)

**What this proves:** “database healthy” is not “application working.” osTicket was looping because auth against MariaDB was wrong.

![Volume recreate then healthy stack](../evidence/screenshots/Container-healthy.png)

**What this proves:** changing `.env` after first boot is not enough. The volume had to be destroyed so MariaDB would create the `osticket` user with the intended password.

### Troubleshooting / lessons learned

**Hypothesis 1:** images failed to pull.  
Rejected: `Pulled` / `Created` succeeded.

**Hypothesis 2:** MariaDB was not ready.  
Rejected: healthcheck passed and logs showed `ready for connections`. There is an `io_uring` / `EPERM` warning on Docker Desktop for Mac. That is a sysctl limitation inside the VM. MariaDB falls back to libaio. It was noise, not the outage.

**Hypothesis 3:** secrets never reached the containers.  
Confirmed: `docker compose config` showed `MARIADB_PASSWORD: ""` and empty `OST_ADMIN_*` values.

**Hypothesis 4:** supplying `.env` and restarting is enough.  
Rejected: MariaDB only applies `MARIADB_USER` / `MARIADB_PASSWORD` when the data directory is empty. The volume already existed. osTicket kept restarting until `docker compose down -v`.

That last point is the one that shows up in production: a stateful service’s *init-time* config is not the same as its *runtime* config. Restarting a pod does not rewrite the user that was created on first start.

**Current residual:** the osTicket container can report Docker `unhealthy` while Apache still returns HTTP 200 on `/scp/`. Healthcheck failure is not the same as user-visible downtime. I would not tell a stakeholder “the help desk is down” from `docker compose ps` alone, and I would not tell them it is fine from a 200 alone.

### Security / operational considerations

- osTicket logs the admin password at install time. That is a product behaviour, not something I configured. In a real environment that log line is an incident.
- `docker compose config` is a diagnostic tool that dumps secrets. Do not paste it into a ticket or a public README.
- Binding to localhost is the blast-radius control for this lab. It is also why a colleague on another machine cannot “just open the help desk.”

### Production delta

- Secrets in a manager, not `.env` on disk.
- Image pin by digest; decide whether a third-party osTicket image is acceptable at all.
- TLS, reverse proxy, session cookie `Secure`, and no install-time password in logs.
- Backups of `osticket-db` and `osticket-attachments` *tested* with a restore, not assumed because a volume exists.
- An app-level healthcheck that hits a login page or `/api`, not only “the process started.”

### Interview talking point

Containers coming up is not the same as the application being usable. MariaDB was healthy with a blank password because it initialized the volume on first boot. osTicket then looped on auth. The fix was not restart — it was destroying the volume so the database user was created with the intended password.
