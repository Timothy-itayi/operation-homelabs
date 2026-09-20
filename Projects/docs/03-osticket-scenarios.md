# 03 — osTicket ticket scenarios

### Objective

Use the live help desk as a queue, not a screenshot of a login page. Create fake employees, file realistic requests, work them with agent behaviour (claim, diagnose, reply, transfer, close), and leave a thread a supervisor could audit.

### Why this matters

Hiring managers do not care that osTicket installed. They care whether I know the difference between:

- a **password** ticket (identity proofing, no secrets in the thread),
- an **access** ticket (approval, least privilege, close with the decision),
- a **VPN** ticket (isolate local network vs corporate concentrator, write the actual cause),
- a **printer** ticket (scope: one device vs floor vs server).

That is Help Desk / Desktop / Cloud Support work. The Compose stack is only the platform.

### What I implemented

**Users (directory, not just tickets):**

| User | Mailbox | Role in the lab |
| --- | --- | --- |
| Alice Saunders | `alice.saunders@homelab.com` | Password lockout after a change |
| Ben Flinder | `ben.flinder@homelab.com` | Access to Finance share for month-end |
| Jessie Lemons | `Jessie.Lemons@homelab.com` | Office printer offline |
| Timothy Itayi | lab end-user (personal mailbox used in the UI — crop before publishing) | VPN from home; later a recurrence |

**Help topics used:** Password / Account Access, VPN / Remote Access, Report a Problem / Access Issue, General IT Support.

**Departments used:** Support, then Network Support on the VPN ticket after triage.

**Queue at the time of this write-up:**

| Ticket | Subject | Requester | Priority | Status |
| --- | --- | --- | --- | --- |
| 344321 | Unable to connect to VPN | Timothy Itayi | High → Normal | **Resolved** (closed by Admin) |
| 795910 | Cannot sign in after password change | Alice Saunders | Emergency | **Resolved** |
| 983225 | Need access to Finance shared folder | Ben Flinder | High | **Resolved** |
| 326898 | VPN will not connect | Timothy Itayi | High | **Open** (recurrence, unassigned) |
| 948476 | Office printer shows offline | Jessie Lemons | Normal | **Open** (unassigned) |
| 511913 | osTicket Installed! | osTicket Team | — | Open (installer leftover) |

---

## Scenario A — VPN timeout from home (the interview ticket)

**Ticket:** `#344321` — Unable to connect to VPN  
**Help topic:** VPN / Remote Access  
**Department:** started in Support, transferred to Network Support  
**Source:** Web

**Intake (user):**

> VPN worked yesterday. Today I receive a connection timeout from home Wi-Fi.

That sentence is enough to start a real workflow. “Worked yesterday” rules out “never enrolled.” “Home Wi-Fi” puts the fault domain on the remote network first, not the office LAN.

**How I worked it:**

1. **Claimed** the ticket so two agents would not duplicate it.
2. Wrote an **internal note** in cause / action / prevention form — the part a supervisor reads, not the user:
   - Cause: home router had stale DNS state after ISP reconnect.
   - Action: confirmed internet connectivity, flushed DNS cache, re-established VPN.
   - Prevention: restart the router if the symptom returns; escalate if it repeats.
3. **Dropped priority High → Normal** after confirming the user’s internet still worked. A timeout with working internet is not “the company is offline.”
4. **Transferred** to Network Support once it was a remote-access/DNS issue rather than a generic “I can’t work” ticket.
5. Sent a **customer reply** that does not dump the internal note. The user gets: internet confirmed, DNS cleared, VPN connecting, what to do if it returns.
6. Closed as **Resolved**.

**Why this is a workflow, not a click-path:** the user-facing reply and the internal note are different documents on purpose. One is for the employee. One is for the next technician. Priority and department changed because the facts changed. That is how a desk stays honest.

**Verification:** thread shows claim → internal note → priority change → department transfer → reply → closed by Admin with status Resolved. Close date `9/18/26 6:08 AM`, about eleven minutes after create — which is only believable because this is a lab. In production I would still want the same *shape* of notes, not the same elapsed time.

**Add screenshot:** `evidence/screenshots/ticket-344321-resolved.png`  
What it must prove: Network Support, internal cause/action/prevention, customer reply, status Resolved. Crop any personal mailbox.

**Production delta:** a real VPN ticket needs the client version, split-tunnel vs full-tunnel, whether other users are affected (concentrator vs one laptop), and a check against the monitoring tool I have not deployed yet. Closing in eleven minutes without a packet capture is fine for a homelab story; I would not pretend it is a network postmortem.

---

## Scenario B — Password change, cannot sign in

**Ticket:** `#795910` — Cannot sign in after password change  
**Help topic:** Password / Account Access  
**Priority:** Emergency  
**Status:** Resolved

**Intake (Alice Saunders):**

> I changed my password yesterday and now I cannot log in.

**How I worked it:** this is identity, not “reset and pray.” Emergency is correct for a user who cannot sign in at all. The resolution reply resets the account and tells the user to collect the temporary credential **through the approved secure channel**, then change it at next login.

I did **not** put a password in the ticket thread. That is the whole point of this scenario. Tickets get exported, screenshotted, and read by people who should not have the credential. The reset belongs in [LLDAP](04-lldap-identity.md). I practiced that action on `ben.flinder`; Alice’s directory object exists but I have not yet captured her `password_modified_date` against this ticket. Same procedure, incomplete audit link.

**Verification:** thread has the user symptom and an agent reply; status Resolved; closed by Admin.

**Add screenshot:** `evidence/screenshots/ticket-795910-password-resolved.png`  
What it must prove: Emergency priority, Password / Account Access topic, reply with no password in the body, status Resolved.

**Production delta:** proof of identity (manager callback, MFA, HR photo), account lock vs bad password vs expired, and an IdP audit event tied to *this* user, not a convenient neighbour in the directory.

---

## Scenario C — Access to Finance shared folder

**Ticket:** `#983225` — Need access to Finance shared folder  
**Requester:** Ben Flinder  
**Help topic:** Report a Problem / Access Issue  
**Priority:** High  
**Status:** Resolved

**Intake:**

> My manager asked me to help with month-end reporting.

That is an **access request**, not a break/fix. The business reason is month-end reporting, which is time-bound and Finance-scoped — so High is defensible.

**How I worked it:** acknowledged the request, said I was checking account and access details, and would update once the cause was confirmed or more information was needed. Then the ticket was resolved.

**Honest gap:** the close-out does not record manager approval, the group that was granted, or a least-privilege check. Ben’s LLDAP record also has **no group memberships**. A hiring manager who has done IAM will notice. In a real desk I would not close this until the thread said: who approved, what group was added, and when it should be reviewed — and the directory would show that group.

**Add screenshot:** `evidence/screenshots/ticket-983225-access-resolved.png`  
What it must prove: High priority, access topic, requester + manager context, resolution reply.

**Production delta:** ticket catalog item with manager approval, group membership in LLDAP/AD, joiner-mover-leaver review, and no standing access “because month-end.”

---

## Scenario D — Printer offline (still open)

**Ticket:** `#948476` — Office printer shows offline  
**Requester:** Jessie Lemons  
**Help topic:** General IT Support  
**Priority:** Normal  
**Status:** Open, unassigned

**Intake:**

> I can print to PDF but not to the office printer.

That one sentence already splits the fault domain: the user’s print stack works (PDF), the physical queue or device does not. Next checks would be: one user vs the floor, printer IP/queue, spooler, and whether the device is actually offline.

Leaving it **open** is correct. Closing it without a device check would be fake.

**Add screenshot:** `evidence/screenshots/ticket-948476-printer-open.png`  
What it must prove: Normal priority, open/unassigned, user statement about PDF vs office printer.

---

## Scenario E — VPN recurrence (still open)

**Ticket:** `#326898` — VPN will not connect  
**Same requester as #344321**  
**Help topic:** VPN / Remote Access  
**Priority:** High  
**Status:** Open, unassigned  
**Due:** Default SLA, `9/21/26 5:00 PM`

**Intake:**

> Internet works, but company VPN times out

This is the follow-up the first ticket’s prevention step predicted. Same symptom class, already isolated to “internet works.” The previous ticket told the user to restart the router and come back if it continued. They came back.

**Why I left it open:** a recurrence is not a duplicate to silently close. It is a signal that the first cause was incomplete or the fix did not persist. Next pass: confirm they did the router restart, check whether other users are affected, then escalate beyond “flushed DNS on one laptop.”

**Add screenshot:** `evidence/screenshots/ticket-326898-vpn-open.png`  
What it must prove: High, unassigned, same user as the resolved VPN ticket, SLA due date visible.

---

### Procedure (repeatable agent path)

1. Create the user in the directory (or take the web intake).
2. Open the ticket with a **help topic** that matches the work type, not “General” for everything.
3. Set priority from **impact**, not from how loudly the user wrote.
4. Claim. If it is the wrong team, transfer — do not work it under the wrong department forever.
5. Internal note = cause / action / prevention. Reply = what the user needs to do or what changed for them.
6. Resolve when the user can work, not when I am tired of the ticket.
7. Leave recurrences open until the first fix is proven insufficient or complete.

### Verification

- Staff console queue shows Open vs Closed counts that match the table above.
- Closed tickets show **Closed By: Admin Admin**.
- User directory lists the fake employees, not a single admin account filing everything as itself.
- At least one ticket has an internal note that is *not* the customer reply.

### Evidence to capture

Drop files under `Projects/evidence/screenshots/` using the names in each scenario. Also capture:

| File | What it must prove |
| --- | --- |
| `queue-open.png` | Open tickets 326898, 948476, leftover installer ticket. |
| `queue-closed.png` | 795910, 983225, 344321 closed by Admin. |
| `users-directory.png` | Alice, Ben, Jessie, plus the VPN requester. No real personal mailbox in frame if this repo goes public. |

### Troubleshooting / lessons learned

The installer creates `#511913 osTicket Installed!` and leaves it open. That is clutter. In a real desk I would close or delete it so the open queue is only user work.

I used a personal mailbox on the VPN tickets. That is fine on a destroyed laptop lab and sloppy in a public GitHub screenshot. Crop it.

osTicket “Resolved” vs “Closed” is easy to muddle. The closed queue is where I proved the work finished. Resolved without a customer-visible reply is how tickets become “why did you close this?”

### Security / operational considerations

- No passwords in ticket bodies.
- Access work needs an approval record before close. I did not fully do that on `#983225`.
- Help topics exist so you do not dump VPN, passwords, and printers into one undifferentiated pile — and so reporting later is not fiction.

### Production delta

- SLAs backed by a real business hours calendar, not a default due date that happens to land on the next Monday.
- Canned responses that match SOPs, not a one-off paragraph typed from memory.
- Identity actions performed in LLDAP/IdP and linked from the ticket, not role-played in prose.
- Queue ownership: nothing user-facing stays unassigned overnight.

### Interview talking point

The VPN ticket is the story: I claimed it, wrote cause/action/prevention internally, dropped the priority once internet was confirmed, moved it to Network Support, and told the user what changed without pasting the internals. When the same user came back with the same class of failure, I left the new ticket open instead of pretending the first close still counted.
