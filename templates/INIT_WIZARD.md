# BACA Session Initialization Wizard

> This document defines the step-by-step flow Claude follows when creating a new BACA session.
> Trigger: `sessions/` directory has no existing WORKLOG, or user says "start a new cross-device conversation".

---

## Wizard Rules

- Ask only one question at a time; wait for the user's answer before continuing
- All questions must be answered before generating files
- Do not skip any question
- After all answers are collected, generate files and output a summary immediately

---

## Question List (in order)

### Q1 — Local Device ID
```
[Init Wizard 1/7]
Enter an identifier for this device.
Suggested format: {model}-{username}
Examples: PC-ALICE, MACMINI-BOB, LAPTOP-WORK

Local Device ID:
```

### Q2 — Remote Device ID
```
[Init Wizard 2/7]
Enter the identifier for the other device (same format).

Remote Device ID:
```

### Q3 — Session Topic Name
```
[Init Wizard 3/7]
Enter a topic name for this session.
This will be used in filenames — use English or hyphenated text.
Examples: stock-workflow, api-design, hardware-comparison

Topic name:
```

### Q4 — Allowed Discussion Scope
```
[Init Wizard 4/7]
List the topics that are allowed in this session (comma-separated).
Example: API design, data formats, scheduling logic, error handling

Allowed scope:
```

### Q5 — Forbidden Discussion Scope
```
[Init Wizard 5/7]
List topics that are explicitly off-limits (comma-separated).
Example: personal accounts, off-topic discussions, financial advice

Forbidden scope:
```

### Q6 — Response Mode
```
[Init Wizard 6/7]
Choose a response mode:

  1. MANUAL (default)
     Agent waits for explicit human instruction before replying.
     Best for: early-stage POC, high human oversight

  2. TIMED-3min
     Agent checks the WORKLOG every 3 minutes automatically.
     Best for: short-interval testing

  3. TIMED-5min
     Agent checks the WORKLOG every 5 minutes automatically.
     Best for: standard testing

  4. TIMED-custom
     Enter a custom interval in minutes.

Enter option (1-4):
```

If option 4 is selected, ask:
```
[Init Wizard 6b/7]
Enter the custom polling interval in minutes (suggest starting with 3):
```

### Q7 — Autonomous Turn Limit
```
[Init Wizard 7/7]
Enter the maximum number of autonomous rounds.
Definition: 1 round = both agents write once (A writes + B writes = 1 round).
When the limit is reached, the session pauses and waits for human authorization.

Start with 3–5 rounds for testing.
Default: 5

Turn limit (press Enter to use default 5):
```

---

## After All Answers Are Collected

### Step 1: Resolve variables

| Variable | Source |
|---|---|
| `DEVICE_A_ID` | Q1 answer |
| `DEVICE_B_ID` | Q2 answer |
| `TOPIC` | Q3 answer |
| `ALLOWED_SCOPE` | Q4 answer |
| `FORBIDDEN_SCOPE` | Q5 answer |
| `RESPONSE_MODE` | Q6 answer (MANUAL / TIMED-3min / TIMED-5min / TIMED-{N}min) |
| `RESPONSE_INTERVAL` | Derived from Q6 (for MANUAL: "human notification") |
| `MAX_AUTO_TURNS` | Q7 answer (default 5 if blank) |
| `CREATED_DATE` | Current date YYYY-MM-DD |
| `CREATED_TIME` | Current time HH:MM |

### Step 2: Generate HANDSHAKE document

- Source template: `templates/HANDSHAKE_PROTOCOL.template.md`
- Target path: `sessions/HANDSHAKE_{DEVICE_A_ID}_{DEVICE_B_ID}_{TOPIC}.md`
- Replace all `{{PLACEHOLDER}}` with actual values

### Step 3: Generate WORKLOG document

- Source template: `templates/WORKLOG.template.md`
- Target path: `sessions/WORKLOG_{DEVICE_A_ID}_{DEVICE_B_ID}_{TOPIC}.md`
- Replace all `{{PLACEHOLDER}}` with actual values
- Initial session state: `INIT_PENDING_B_CONFIRM`

### Step 4: Output summary to user

```
✅ Initialization complete!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files created:
  📋 sessions/HANDSHAKE_{A-ID}_{B-ID}_{topic}.md
  📝 sessions/WORKLOG_{A-ID}_{B-ID}_{topic}.md

Session settings:
  Topic: {TOPIC}
  Response mode: {RESPONSE_MODE}
  Turn limit: {MAX_AUTO_TURNS}
  Session state: INIT_PENDING_B_CONFIRM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next steps:

  1. Confirm the files have synced to the other device
     (OneDrive / iCloud / Dropbox / Syncthing)

  2. Notify the user on {DEVICE_B_ID}:
     "Please open Claude Code Desktop, navigate to this project,
      read WORKLOG_{A-ID}_{B-ID}_{topic}.md and write a confirmation entry"

  3. Wait for confirmation:
     • MANUAL mode: tell me "Device B has confirmed" when ready
     • TIMED mode: I will check the WORKLOG every {N} minutes automatically

Now waiting for: {DEVICE_B_ID} to confirm the protocol
```

---

## Device B Confirmation Flow

When Device B's Claude reads a WORKLOG with session state `INIT_PENDING_B_CONFIRM`:

1. Read `sessions/HANDSHAKE_{A}_{B}_{topic}.md`
2. Read `sessions/WORKLOG_{A}_{B}_{topic}.md`
3. Confirm understanding of all settings (topic, scope, mode, limit)
4. Append a confirmation entry (format below)
5. Update the WORKLOG header `Session State` field to `ACTIVE`

### Device B Confirmation Entry Format

```markdown
## YYYY-MM-DD HH:MM | {DEVICE_B_ID} | Protocol Confirmation

### ✅ Pre-write Checklist
- [x] Responded to previous Action Items? → Yes
- [x] Content within topic scope? → Yes (this is a confirmation entry, part of init flow)
- [x] Verification result: PASS

### 📥 Confirming {DEVICE_A_ID}'s Action Items
- [x] [{DEVICE_B_ID}] Read HANDSHAKE_PROTOCOL.md ✅
- [x] [{DEVICE_B_ID}] Read WORKLOG and confirmed all settings ✅
- [x] [{DEVICE_B_ID}] Wrote confirmation entry (this entry) ✅

### ✅ Completed This Turn
- Read and understood protocol v1.0
- Confirmed topic: {TOPIC_NAME}
- Confirmed allowed scope: {ALLOWED_SCOPE}
- Confirmed forbidden scope: {FORBIDDEN_SCOPE}
- Confirmed response mode: {RESPONSE_MODE}
- Confirmed turn limit: {MAX_AUTO_TURNS} rounds

### 💬 Message to {DEVICE_A_ID}
Protocol confirmed. Session state updated to ACTIVE.
Please write the first formal conversation entry; turn count starts at Round 1.

### 🟡 State Update
Session state: INIT_PENDING_B_CONFIRM → **ACTIVE**
```

---

## TIMED Mode: /loop Command

If response mode is TIMED, start the auto-polling loop on Device A:

```
/loop {N}m Check sessions/WORKLOG_{A}_{B}_{topic}.md for the latest entry.
If the latest entry is from the counterpart and has Action Items for me,
run the Pre-write Checklist and write a response entry.
If the latest entry is still mine, report "No response yet, still waiting."
If Session State becomes PAUSED_LIMIT_REACHED, stop the loop and notify the user.
```
