# CLAUDE.md — BACA Device A

## This Directory

| Field | Value |
|---|---|
| Role | Device A (session initiator) |
| Local Device ID | PC-ALICE ← change this |
| Remote Device ID | MACMINI-BOB ← change this |
| Created | YYYY-MM-DD |

**Important**: This directory is an isolated BACA working environment.

---

## Shared Sessions Location

WORKLOG and HANDSHAKE files are stored in a **cloud-synced shared folder**:

```
{YOUR_SYNC_PATH}/worklog-sessions/
```

Examples:
- OneDrive: `C:\Users\{username}\OneDrive\worklog-sessions\`
- iCloud: `~/Library/Mobile Documents/com~apple~CloudDocs/worklog-sessions/`
- Dropbox: `~/Dropbox/worklog-sessions/`
- Local (same machine, two sessions): `/tmp/worklog-sessions/`

> Both devices must point to the same synced folder.

---

## Mandatory Steps at Session Start

### Step 1: Scan the sessions folder

Check whether the shared sessions path contains existing WORKLOG files.

**Case A — No WORKLOG files:**
→ Tell the user: "No active sessions found. Starting initialization wizard."
→ Run the initialization wizard from `templates/INIT_WIZARD.md`
→ Write generated files to the shared sessions path above

**Case B — WORKLOG files exist:**
→ Read the latest WORKLOG
→ Execute the flow corresponding to its session state:

| State | Action |
|---|---|
| `INIT_PENDING_B_CONFIRM` | Remind user: waiting for Device B to confirm; do not write formal entries |
| `ACTIVE` | Read last 3 entries → confirm turn count → run Pre-write Checklist → continue |
| `PAUSED_LIMIT_REACHED` | Notify user: turn limit reached; human authorization required |
| `CLOSED` | Notify user: session closed; ask if they want to start a new one |

### Step 2: Handle response mode

- **MANUAL**: Wait for user to say "Device B has confirmed, please respond"
- **TIMED-Xmin**: Inform user "Will check WORKLOG every X minutes" and start `/loop`

---

## Template Locations

| File | Path |
|---|---|
| Initialization wizard | `templates/INIT_WIZARD.md` |
| Protocol template | `templates/HANDSHAKE_PROTOCOL.template.md` |
| WORKLOG template | `templates/WORKLOG.template.md` |

---

## Pre-filled Values (Device A)

When running the initialization wizard, these values are pre-set:

- **Local Device ID**: `PC-ALICE` ← replace with your device ID
- **Remote Device ID**: `MACMINI-BOB` ← replace with the other device's ID
