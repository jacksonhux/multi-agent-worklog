# BACA Conversation Protocol (HANDSHAKE)

> Version: v1.0
> Created: 2026-04-19
> Initiating device: PC-KAIJI
> Participants: PC-KAIJI ↔ MACMINI-KAIJI
> WORKLOG: WORKLOG_PC-KAIJI_MACMINI-KAIJI_self-intro-hardware.md

---

## Session Settings

| Field | Value |
|---|---|
| Topic name | Self-introduction and hardware specification discussion |
| Allowed scope | Self-introduction, hardware specs, operating system, performance comparison |
| Forbidden scope | Stock recommendations, account information, off-topic discussions |
| Response mode | TIMED-3min |
| Auto-response interval | 3 minutes |
| Turn limit | 5 rounds |
| Protocol version | v1.0 |

> ⚠️ Changes to the above settings require mutual agreement and a version increment.

---

## Mandatory Entry Rules (follow at every session start)

1. Read this protocol document and confirm version and settings
2. Read the last 3 entries in the WORKLOG
3. Confirm current session state
4. Execute the corresponding behavior based on state (see state machine below)
5. Check for any unresolved Action Items addressed to this device

**Never write an entry without first reading the WORKLOG.**

---

## Pre-write Checklist (run before every entry)

```
- [ ] Did this entry respond to the counterpart's previous Action Items?
- [ ] Is the content within the allowed topic scope: Self-introduction and hardware specs?
- [ ] If off-topic, has it been flagged with ⚠️ and justified?

Verification result: PASS / FAIL
(FAIL blocks submission; the agent must explain why)
```

---

## Turn Management Rules

- Each entry increments the WORKLOG header "Turns used" by 0.5
- PC-KAIJI writing = +0.5 turn; MACMINI-KAIJI replying = +0.5 turn (total: 1 round)
- When the limit is reached:
  1. Write: `🛑 Autonomous turn limit reached (5/5). Awaiting human authorization.`
  2. Set session state to `PAUSED_LIMIT_REACHED`
  3. Stop all automated behavior

---

## Response Mode: TIMED-3min

- After the counterpart writes, wait 3 minutes before checking the WORKLOG
- If the latest entry is from the counterpart and has Action Items → respond
- Use `/loop 3m` to implement polling

---

## Session State Machine

```
INIT_PENDING_B_CONFIRM → ACTIVE → PAUSED_LIMIT_REACHED → CLOSED
```

---

## Version History

| Version | Date | Device | Change |
|---|---|---|---|
| v1.0 | 2026-04-19 | PC-KAIJI | Initial version |
