# multi-agent-worklog

> **Asynchronous multi-agent conversation via a shared Markdown WORKLOG.**
> Zero infrastructure. Human-readable at every step. Works across devices.
>
> Formally: **BACA** (Blackboard-Augmented Conversational Architecture) — see [`BACA_protocol_description.md`](./BACA_protocol_description.md) for the academic treatment.

---

## What is this?

Two Claude Code Desktop sessions — on the **same machine** or on **different devices** — hold a structured, autonomous conversation using nothing but a shared Markdown file synchronized via cloud storage (OneDrive, iCloud, Dropbox, Syncthing, or any file-sync service).

No servers. No message queues. No custom code. Just two agents, one file, and a protocol.

```
Device A (Windows / macOS / Linux)          Device B (macOS / Windows / Linux)
┌──────────────────────┐                    ┌──────────────────────┐
│  Claude Code Desktop │                    │  Claude Code Desktop │
│  + CLAUDE.md (Agent A│                    │  + CLAUDE.md (Agent B│
└──────────┬───────────┘                    └──────────┬───────────┘
           │ read / append                             │ read / append
           └──────────────────┬────────────────────────┘
                              │
                   ┌──────────▼──────────┐
                   │  WORKLOG_{A}_{B}_   │
                   │  {topic}.md         │  ← synced via OneDrive /
                   │  (shared document)  │    iCloud / Dropbox / etc.
                   └─────────────────────┘
```

---

## Key Features

- **Zero infrastructure** — shared file + any cloud sync is all you need
- **Human-readable at every step** — the entire conversation is plain Markdown
- **Built-in governance** — Pre-write Checklist, topic scope, turn limits, state machine
- **Genuinely isolated agents** — each Claude session has no shared hidden state
- **LLM-agnostic** — any LLM that reads Markdown can participate
- **Works same-device too** — two local sessions can use a local path instead of cloud sync

---

## Quick Start

### Step 1 — Set up the shared folder

Create a folder that both devices can access via cloud sync:
```
OneDrive/SharedDrive/worklog-sessions/
```

### Step 2 — Set up Device A (initiator)

1. Clone this repo on Device A
2. Copy `templates/` into your working directory
3. Create a `CLAUDE.md` pointing to the shared sessions folder (see `examples/CLAUDE_device_a.md`)
4. Open Claude Code Desktop in that directory

### Step 3 — Initialize a session

Tell Claude Code:
```
開始新的跨裝置對話
```

Claude will run the 7-question **INIT_WIZARD** and generate:
- `HANDSHAKE_{A}_{B}_{topic}.md` — protocol agreement
- `WORKLOG_{A}_{B}_{topic}.md` — conversation log

Both files are written directly to the shared sync folder.

### Step 4 — Set up Device B (responder)

1. Clone this repo on Device B (or copy `templates/` + a `CLAUDE.md`)
2. Open Claude Code Desktop pointed at the same shared folder
3. Claude will detect `INIT_PENDING_B_CONFIRM` state and run the confirmation flow automatically

### Step 5 — Start the conversation

**Recommended (v1.1): Event-driven mode via `Monitor`**

On each device, arm a `Monitor` watching the WORKLOG for the counterpart's entries:

```
Monitor(
  command: tail -F {WORKLOG_PATH} 2>/dev/null | grep --line-buffered -E "^## .* \| {COUNTERPART_ID} \|",
  persistent: true
)
```

For cross-device setups where the WORKLOG lives on the other machine, wrap `tail -F` in SSH:

```
ssh {user}@{host} "tail -F {REMOTE_WORKLOG_PATH} 2>/dev/null" | grep --line-buffered -E "^## .* \| {COUNTERPART_ID} \|"
```

The counterpart writes a new entry → `tail -F` streams it → `Monitor` emits a `<task-notification>` → Claude wakes and responds. **Measured latency: ~12–15 seconds, same-device or cross-device.**

**Fallback: Polling mode via `/loop`**

If `Monitor` or `tail -F` is not available in your environment:
```
/loop 30s
```
This re-reads the WORKLOG every 30 seconds. Higher and less stable latency than the event-driven mode.

---

## Repository Structure

```
multi-agent-worklog/
├── README.md                          ← this file
├── BACA_protocol_description.md       ← full technical & academic description
├── templates/
│   ├── INIT_WIZARD.md                 ← session initialization wizard
│   ├── HANDSHAKE_PROTOCOL.template.md ← protocol agreement template
│   └── WORKLOG.template.md            ← conversation log template
├── examples/
│   ├── CLAUDE_device_a.md             ← sample CLAUDE.md for Device A
│   ├── CLAUDE_device_b.md             ← sample CLAUDE.md for Device B
│   └── session-01/                    ← complete real-world session example
│       ├── HANDSHAKE_...md
│       ├── WORKLOG_...md
│       └── SUMMARY_...md
└── .gitignore
```

---

## How It Works

### Session Lifecycle

```
INIT_PENDING_B_CONFIRM  →  ACTIVE  →  PAUSED_LIMIT_REACHED  →  CLOSED
```

| State | Meaning |
|---|---|
| `INIT_PENDING_B_CONFIRM` | Waiting for Device B to read and confirm the protocol |
| `ACTIVE` | Conversation in progress |
| `PAUSED_LIMIT_REACHED` | Turn limit reached; human authorization required to continue |
| `CLOSED` | Session complete; WORKLOG is sealed |

### Pre-write Checklist (Governance)

Every entry must pass this checklist before being written:
```
- [ ] Did this entry respond to the previous agent's Action Items?
- [ ] Is the content within the defined topic scope?
- [ ] If off-topic, has it been flagged with ⚠️ and justified?

Verification result: PASS / FAIL
```

### Response Modes

| Mode | Behavior | Typical Latency |
|---|---|---|
| `EVENT` ⭐ (v1.1, recommended) | `Monitor` + `tail -F` wakes the agent on file change | **~12–15 s** |
| `TIMED-Xmin` | Agent polls the WORKLOG every X minutes | 0 – X·60 s |
| `MANUAL` | Agent waits for explicit human instruction | unbounded |

---

## Empirical Validation

A complete proof-of-concept session is included in `examples/session-01/`:

- **Devices**: Windows 11 laptop (Intel Core Ultra 9 185H + RTX 4080) ↔ macOS 26 Mac Mini (Apple M4)
- **Topic**: Hardware self-introduction and cross-platform comparison
- **Duration**: 53 minutes (2026-04-19 23:10 → 2026-04-20 00:03)
- **Rounds**: 4/5 (early closure by mutual agreement)
- **Human interventions per turn**: 0
- **Protocol violations**: 0
- **Pre-write Checklist failures**: 0

Notable finding: In Round 2, the Mac Mini agent **self-corrected** a factual claim from Round 1 after independently running `which ollama` and `python3 -c "import mlx"` — demonstrating that genuine context isolation produces qualitatively different behavior from single-agent role-play.

### v1.1 Event-Driven Validation (2026-04-20)

A follow-up validation sequence confirmed that the `Monitor` + `tail -F` architecture (response mode `EVENT`) delivers consistent sub-15-second wake-up latency across three scenarios:

| Phase | Write source | Transport | End-to-end latency |
|---|---|---|---|
| 1 | Bash script | Local `tail -F` | ~13 s |
| 2 | LLM subagent | Local `tail -F` | ~12 s |
| 3 | Remote Bash on macOS | SSH-tunnelled `tail -F` | ~12–15 s |

This is a **~10× improvement** over v1.0's `TIMED-3min` polling, with no new infrastructure. The binding constraint is Claude Code's own notification-delivery cadence (~12 s); file I/O, network, and subprocess overhead each contribute <1 second. See `BACA_protocol_description.md` §5.3 for the full methodology.

---

## Design Principles

1. **The document is the protocol** — no hidden state, ever
2. **Append-only** — entries are never modified or deleted
3. **Explicit over implicit** — every entry is self-sufficient context
4. **Human legibility first** — any human can read and intervene at any point
5. **Governance in natural language** — rules are Markdown, not code

---

## Comparison with Other Approaches

| Approach | Infrastructure | Human-readable channel | Cross-device | Governance layer |
|---|---|---|---|---|
| **BACA** | File sync only | ✅ Full Markdown | ✅ Native | ✅ Built-in |
| Claude Code Agent Teams | Anthropic platform | ❌ Opaque mailbox | ⚠️ Unclear | ❌ Platform-managed |
| AutoGen / LangGraph | Python server | ❌ API calls | ⚠️ With infra | ⚠️ Framework-level |
| MCP Agent Mail | MCP + SQLite | ❌ Tooling required | ⚠️ With infra | ❌ None built-in |

---

## Academic Context

BACA is a modern realization of the classical **Blackboard Architecture** (Hearsay-II, 1980), adapted for LLM agents. See `BACA_protocol_description.md` for the full technical and academic treatment.

---

## License

MIT — use freely, attribution appreciated.

---

## Citation

```
Hu, K. (2026). BACA: Blackboard-Augmented Conversational Architecture for
Asynchronous Cross-Device LLM Agent Collaboration.
https://github.com/{your-username}/multi-agent-worklog
```
