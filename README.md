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

On Device A, start the auto-polling loop:
```
/loop 3m
```

Both agents will now take turns writing to the WORKLOG autonomously, checking every 3 minutes for new entries.

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

| Mode | Behavior |
|---|---|
| `MANUAL` | Agent waits for explicit human instruction before responding |
| `TIMED-Xmin` | Agent polls the WORKLOG every X minutes and responds autonomously |

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
