# CLAUDE.md — BACA Device B

## This Directory

| Field | Value |
|---|---|
| Role | Device B (session responder) |
| Local Device ID | MACMINI-BOB ← change this |
| Remote Device ID | PC-ALICE ← change this |
| Created | YYYY-MM-DD |

**Important**: This directory is an isolated BACA working environment.

---

## Shared Sessions Location

WORKLOG and HANDSHAKE files are in the **cloud-synced shared folder** (same path as Device A):

```
{YOUR_SYNC_PATH}/worklog-sessions/
```

Examples:
- OneDrive (macOS): `~/Library/CloudStorage/OneDrive-Personal/worklog-sessions/`
- iCloud: `~/Library/Mobile Documents/com~apple~CloudDocs/worklog-sessions/`
- Dropbox: `~/Dropbox/worklog-sessions/`
- Local (same machine): `/tmp/worklog-sessions/`

---

## Mandatory Steps at Session Start

### Step 1: Scan the sessions folder

**Case A — No WORKLOG files:**
→ Tell the user: "No active sessions found. Sessions are initiated from Device A."

**Case B — WORKLOG files exist:**
→ Read the latest WORKLOG and execute based on state:

| State | Action |
|---|---|
| `INIT_PENDING_B_CONFIRM` | **Run Device B confirmation flow immediately** (see below) |
| `ACTIVE` | Read last 3 entries → confirm turn → Pre-write Checklist → continue |
| `PAUSED_LIMIT_REACHED` | Notify user: turn limit reached; awaiting human authorization |
| `CLOSED` | Notify user: session closed; ask if they want to start a new one |

### Step 2: Handle response mode

Current setting: check HANDSHAKE document for `RESPONSE_MODE`.

- **MANUAL**: Wait for user to say "Please respond"
- **TIMED-Xmin**: Start `/loop Xm` to poll automatically

---

## Device B Confirmation Flow (when state = INIT_PENDING_B_CONFIRM)

1. Read the HANDSHAKE document
2. Read the WORKLOG; confirm understanding of all settings
3. Append a confirmation entry in this format:

```markdown
## YYYY-MM-DD HH:MM | {YOUR_DEVICE_ID} | Protocol Confirmation

### ✅ Pre-write Checklist
- [x] Responded to previous Action Items? → Yes
- [x] Content within scope? → Yes (confirmation entry; part of init flow)
- [x] Verification result: PASS

### 📥 Confirming Device A's Action Items
- [x] [{YOUR_DEVICE_ID}] Read HANDSHAKE_PROTOCOL.md ✅
- [x] [{YOUR_DEVICE_ID}] Read WORKLOG and confirmed all settings ✅
- [x] [{YOUR_DEVICE_ID}] Wrote confirmation entry (this entry) ✅

### ✅ Completed This Turn
- Read and understood protocol v1.0
- Confirmed topic: {TOPIC_NAME}
- Confirmed allowed scope: {ALLOWED_SCOPE}
- Confirmed forbidden scope: {FORBIDDEN_SCOPE}
- Confirmed response mode: {RESPONSE_MODE}
- Confirmed turn limit: {MAX_AUTO_TURNS} rounds

### 💬 Message to Device A
Protocol confirmed. Session state updated to ACTIVE.
Please write the first formal entry; turn count starts at Round 1.

### 🟡 State Update
Session state: INIT_PENDING_B_CONFIRM → **ACTIVE**
```

4. Update the WORKLOG header `Session state` field to `ACTIVE`

---

## Template Locations

| File | Path |
|---|---|
| Initialization wizard | `../templates/INIT_WIZARD.md` |
| Protocol template | `../templates/HANDSHAKE_PROTOCOL.template.md` |
| WORKLOG template | `../templates/WORKLOG.template.md` |
