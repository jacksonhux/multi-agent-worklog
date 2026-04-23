# BACA Conversation Protocol (HANDSHAKE)

> Version: v1.0
> Created: {{CREATED_DATE}}
> Initiating device: {{DEVICE_A_ID}}
> Participants: {{DEVICE_A_ID}} ↔ {{DEVICE_B_ID}}
> WORKLOG: WORKLOG_{{DEVICE_A_ID}}_{{DEVICE_B_ID}}_{{TOPIC}}.md

---

## Session Settings

| Field | Value |
|---|---|
| Topic name | {{TOPIC_NAME}} |
| Allowed scope | {{ALLOWED_SCOPE}} |
| Forbidden scope | {{FORBIDDEN_SCOPE}} |
| Response mode | {{RESPONSE_MODE}} |
| Auto-response interval | {{RESPONSE_INTERVAL}} |
| Turn limit | {{MAX_AUTO_TURNS}} rounds |
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

Before writing any entry, complete all three checks. Only proceed if all pass:

```
- [ ] Did this entry respond to the counterpart's previous Action Items?
- [ ] Is the content within the allowed topic scope: {{TOPIC_NAME}}?
- [ ] If off-topic, has it been flagged with ⚠️ and justified?

Verification result: PASS / FAIL
(FAIL blocks submission; the agent must explain why)
```

---

## Turn Management Rules

- Each entry increments the WORKLOG header "Turns Used" by 0.5
- Device A writing = +0.5 turn; Device B replying = +0.5 turn (total: 1 round)
- When the limit is reached:
  1. Write: `🛑 Autonomous turn limit reached ({{MAX_AUTO_TURNS}}/{{MAX_AUTO_TURNS}}). Awaiting human authorization.`
  2. Set session state to `PAUSED_LIMIT_REACHED`
  3. Stop all automated behavior; wait for human instruction

---

## Response Mode Reference

### MANUAL Mode
- Agent does **not** respond automatically
- Waits for explicit user instruction: "Device B has confirmed, please respond"
- Best for: early-stage POC, high human oversight per round

### TIMED Mode
- After the counterpart writes, wait {{RESPONSE_INTERVAL}} before checking the WORKLOG
- If the latest entry is from the counterpart and has Action Items → run Pre-write Checklist → respond
- If the latest entry is from this device → continue waiting
- Use `/loop` to implement periodic polling

**Current mode: {{RESPONSE_MODE}}**

---

## Session State Machine

```
INIT_PENDING_B_CONFIRM
        │
        │  Device B writes confirmation entry → ACTIVE
        ▼
      ACTIVE ◄────────────────────────────┐
        │                                 │
        │  Turn limit reached             │ Human authorization
        ▼                                 │
  PAUSED_LIMIT_REACHED ──────────────────►┘
        │
        │  Human declares closure
        ▼
     CLOSED
```

| State | Meaning | Who can change |
|---|---|---|
| `INIT_PENDING_B_CONFIRM` | Waiting for Device B to confirm; no formal entries allowed | Device B changes to ACTIVE |
| `ACTIVE` | Conversation in progress | Either device (per rules) |
| `PAUSED_LIMIT_REACHED` | Turn limit reached; paused for human authorization | Either device after human auth |
| `CLOSED` | Conversation complete; WORKLOG is sealed | Either device; irreversible |

---

## Entry Format Standard

### Header (required)
```
## YYYY-MM-DD HH:MM | {DEVICE_ID} | Entry Topic
```

### Content Blocks

| Block | Purpose | When required |
|---|---|---|
| `✅ Pre-write Checklist` | Three self-verification items | **Every entry** |
| `📥 Confirm Previous Action Items` | Status of counterpart's requests | When counterpart has pending items |
| `✅ Completed This Turn` | Factual statements of what was done | **Every entry** |
| `🔧 Technical Action Log` | Commands, outputs, results | When technical actions were taken |
| `🧭 Decision Record` | Decisions finalized this session | When decisions are made |
| `📂 File Changes` | Full absolute paths of modified/created files | When files were changed |
| `🔴 Action Items for Counterpart` | Specific tasks assigned to the other device | When the counterpart needs to do something |
| `💬 Questions / Pending` | Questions needing the counterpart's answer | When clarification is needed |

### Action Items Format
```
- [ ] [{{DEVICE_A_ID}}] Task description (clear, actionable)
- [ ] [{{DEVICE_B_ID}}] Task description (clear, actionable)
```

---

## Status Markers

| Marker | Meaning |
|---|---|
| 🔴 | Needs counterpart action; blocks progress |
| 🟡 | FYI for counterpart; not urgent |
| ✅ | Completed / confirmed |
| 📌 | Planned; not yet executed |
| ❓ | Open question; needs discussion |
| ⚠️ | Caution; may affect counterpart |
| 🛑 | Stop signal; human intervention needed |

---

## Anti-Patterns (Forbidden)

| Anti-pattern | Correct approach |
|---|---|
| Modifying or deleting previous entries | Append-only; correct errors in a new entry |
| Omitting timestamps | Always include precise timestamps (YYYY-MM-DD HH:MM) |
| Using relative paths | Always use absolute paths |
| Action Items without device assignment | Must include `[DEVICE_ID]` |
| Mixing facts with plans | ✅ Completed / 📌 Planned — distinguish clearly |
| Assuming counterpart has context | Each entry must be self-sufficient |
| Writing without reading | Always read the last 3 entries first |
| Off-topic without flagging | Must mark with ⚠️ and justify |
| Skipping Pre-write Checklist | Must run every time; PASS required to submit |

---

## Version History

| Version | Date | Device | Change |
|---|---|---|---|
| v1.0 | {{CREATED_DATE}} | {{DEVICE_A_ID}} | Initial version |
