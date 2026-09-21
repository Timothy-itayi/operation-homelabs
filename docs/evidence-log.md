# Evidence log

Each image is linked from the write-up. This table is the index.

## Docker / first boot

| File | Proves |
| --- | --- |
| `screenshots/Docker_Stack_Running.png` | osTicket Compose project in Docker Desktop; `8080:80`. |
| `screenshots/MariaDB-Rejects-Auth.png` | Unset `OST_*` vars interpolating as blank strings. |
| `screenshots/resolve-Auth-Issue.png` | MariaDB healthy, osTicket `Restarting`. |
| `screenshots/Container-healthy.png` | `down -v`, recreate, localhost publish. |

## osTicket

| File | Proves |
| --- | --- |
| `screenshots/osTicket/osticket-authPage.png` | Staff login on `localhost:8080`. |
| `screenshots/osTicket/Unable-to connect to VPN-TIcket-Resolved.png` | VPN `#344321` thread: claim, note, transfer, reply. |
| `screenshots/osTicket/Finance-Access-control-user-ticket.png` | Finance access `#983225` acknowledgement. |
| `screenshots/osTicket/Office-printer-ticket.png` | Printer `#948476` still open after ack. |

## LLDAP

| File | Proves |
| --- | --- |
| `screenshots/LLDAP/LLDAP-user-list.png` | Lab users under `@homelab.internal`. |
| `screenshots/LLDAP/LLDAP-USER_PWR.png` | `ben.flinder` object + Modify password. |
| `screenshots/LLDAP/LLDAP-resolve-USer_PasswordREset.png` | Alice password ticket `#795910` — reset reply, no secret in the thread. |

## Uptime Kuma

| File | Proves |
| --- | --- |
| `screenshots/KumaMonitor/KumaDashboard.png` | Fresh dashboard, no monitors yet. |
| `screenshots/KumaMonitor/StatusPage-ForLocalServices.png` | `/status/labs` with Azure, LLDAP, osTicket. |
| `screenshots/KumaMonitor/upTimeKuma-Server-Failure.png` | osTicket DOWN, `ECONNREFUSED`. |
| `screenshots/KumaMonitor/upTimeKuma-Server-Restart.png` | osTicket UP, `200 - OK`. |
| `screenshots/KumaMonitor/HelpDesk-Outage-Ticket.png` | Outage ticket `#765614` opened. |
| `screenshots/KumaMonitor/HelpDesk-Outage-tickerREsolved.png` | Internal note: detect / diagnose / resolve / validate. |
