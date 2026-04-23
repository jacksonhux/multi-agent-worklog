# WORKLOG — {{DEVICE_A_ID}} ↔ {{DEVICE_B_ID}}

> Protocol: [HANDSHAKE_{{DEVICE_A_ID}}_{{DEVICE_B_ID}}_{{TOPIC}}.md](./HANDSHAKE_{{DEVICE_A_ID}}_{{DEVICE_B_ID}}_{{TOPIC}}.md)
> **Rule: Append-only. Never modify or delete existing entries.**

---

## Session Info (maintained by agents; update on state changes)

| Field | Value |
|---|---|
| Session ID | {{DEVICE_A_ID}} ↔ {{DEVICE_B_ID}} |
| Topic | {{TOPIC_NAME}} |
| Allowed scope | {{ALLOWED_SCOPE}} |
| Forbidden scope | {{FORBIDDEN_SCOPE}} |
| Initiating device | {{DEVICE_A_ID}} |
| Created | {{CREATED_DATE}} {{CREATED_TIME}} |
| Response mode | {{RESPONSE_MODE}} |
| Turn limit | {{MAX_AUTO_TURNS}} rounds |
| **Turns used** | **0 / {{MAX_AUTO_TURNS}}** |
| **Session state** | **INIT_PENDING_B_CONFIRM** |

> Turn calculation: Device A writes = +0.5; Device B replies = +0.5; total 1 round.
> Update the fields above whenever state changes.

---

## {{CREATED_DATE}} {{CREATED_TIME}} | {{DEVICE_A_ID}} | Session Initialization

### ✅ Pre-write Checklist
- [x] Responded to previous Action Items? → N/A (this is the first entry)
- [x] Content within topic scope? → Yes (initialization is a system flow)
- [x] Verification result: **PASS**

### ✅ Completed This Turn
- Ran initialization wizard; answered all 7 setup questions
- Generated HANDSHAKE_PROTOCOL document
- Generated this WORKLOG document
- Set session state to `INIT_PENDING_B_CONFIRM`

### 🧭 Session Settings Record
- Topic: {{TOPIC_NAME}}
- Allowed scope: {{ALLOWED_SCOPE}}
- Forbidden scope: {{FORBIDDEN_SCOPE}}
- Response mode: {{RESPONSE_MODE}}
- Turn limit: {{MAX_AUTO_TURNS}} rounds

### 🔴 Action Items for {{DEVICE_B_ID}}
- [ ] [{{DEVICE_B_ID}}] Read the HANDSHAKE_PROTOCOL document
- [ ] [{{DEVICE_B_ID}}] Read this WORKLOG and confirm understanding of all settings
- [ ] [{{DEVICE_B_ID}}] Append a confirmation entry and set session state to `ACTIVE`

### 💬 Message to {{DEVICE_B_ID}}
This is the initialization record for our cross-device BACA session.
Files are synced via cloud storage — you can read them directly on your device.
After reading the protocol, append your confirmation entry to officially start the session.

---

<!--
════════════════════════════════════════════════════════════
{{DEVICE_B_ID}} appends confirmation entry below.
See templates/INIT_WIZARD.md for the required format.
════════════════════════════════════════════════════════════
-->

<!-- Device B appends here; do not remove this comment -->

---

<!--
════════════════════════════════════════════════════════════
Once session state is ACTIVE, formal conversation entries begin here.
Each entry must include:
  1. Header: ## YYYY-MM-DD HH:MM | Device ID | Entry Topic
  2. ✅ Pre-write Checklist (required every entry)
  3. ✅ Completed This Turn (required every entry)
  4. Other blocks as needed
════════════════════════════════════════════════════════════
-->
