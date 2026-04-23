# BACA: Blackboard-Augmented Conversational Architecture for Asynchronous Cross-Device LLM Agent Collaboration

**Kaiji Hu**
Independent Research · April 2026

---

## Abstract

We present **BACA** (Blackboard-Augmented Conversational Architecture), a lightweight protocol enabling asynchronous, structured collaboration between independent Large Language Model (LLM) instances running on physically separate devices. Unlike existing multi-agent frameworks that require dedicated infrastructure (message queues, orchestration servers, or proprietary APIs), BACA uses a single append-only structured Markdown document synchronized via commodity cloud storage as its sole communication medium. The protocol incorporates a governance layer comprising topic scope enforcement, pre-write verification checklists, a finite state machine for session lifecycle management, and a configurable turn-limit mechanism for human oversight. A proof-of-concept validation conducted between a Windows 11 laptop (PC-KAIJI, Intel Core Ultra 9 185H + RTX 4080) and a macOS 26 Mac Mini (MACMINI-KAIJI, Apple M4) demonstrated successful four-round autonomous collaboration with zero human intervention per turn, completing a structured hardware specification exchange and comparative analysis in under one hour. We argue that BACA's primary contribution is not performance but **auditability, transparency, and infrastructure independence**: properties that become critical as LLM agents move from research prototypes to everyday collaborative tools.

---

## 1. Introduction

The deployment of LLM agents in multi-agent settings has accelerated considerably. Frameworks such as AutoGen [1], LangGraph [2], and Anthropic's experimental Claude Code Agent Teams provide coordination primitives for parallel task execution. However, these solutions share a common architectural assumption: coordination infrastructure is managed by the platform, hidden from the end user, and typically optimized for throughput over interpretability.

This assumption creates three practical problems:

1. **Opacity**: Communication between agents occurs through binary or JSON channels that are machine-readable but not human-inspectable without specialized tooling.
2. **Infrastructure dependency**: Deploying multi-agent systems requires engineers to provision message brokers, orchestration layers, or proprietary cloud services.
3. **Context conflation**: In systems where a single LLM "role-plays" multiple agents, the claimed independence of agents is simulated rather than genuine.

BACA addresses all three problems through a different architectural philosophy: **the communication medium is the artifact**. A single Markdown file — human-readable, version-controllable, and synchronizable via any cloud storage service — serves as both the message transport and the persistent audit log. Agents are genuinely isolated: each runs in an independent session on a separate physical device with no shared context beyond what is explicitly written in the shared document.

This paper describes the BACA protocol, its design rationale, its empirical validation, and its positioning relative to the existing literature.

---

## 2. Background and Related Work

### 2.1 Blackboard Architecture

BACA draws direct inspiration from the Blackboard architectural pattern first formalized in the Hearsay-II speech understanding system [3]. In the classical blackboard model, a shared data structure (the "blackboard") is the sole communication medium between independent knowledge sources (KSs). Each KS reads from and writes to the blackboard; no KS has direct knowledge of the others' internal state.

BACA modernizes this pattern for LLM agents: the WORKLOG document is the blackboard, and each Claude Code Desktop session is a knowledge source. The critical innovation is that the blackboard is structured natural language rather than a domain-specific data schema, making it simultaneously machine-processable (by the LLM) and human-readable.

### 2.2 Event Sourcing for Agents

Recent work on ESAA (Event Sourcing for Autonomous Agents) [4] proposes an append-only event log as the authoritative record of agent actions. BACA independently converges on the same append-only constraint — the WORKLOG protocol explicitly forbids modification or deletion of existing entries — for the same reason: immutability enables reliable state reconstruction, debugging, and auditing.

### 2.3 File-Based Agent Communication

The closest practical precedents to BACA are:

- **MCP Agent Mail** [5]: A mailbox-style coordination layer using SQLite indexing and Git backup, supporting file lease systems for conflict prevention. More featureful than BACA but requires MCP infrastructure.
- **Agent Message Queue** [6]: A Maildir-inspired zero-infrastructure file queue supporting atomic delivery. Lacks BACA's bidirectional conversation semantics and governance layer.
- **TASKS.md pattern** [7]: Lightweight unidirectional task delegation via Markdown files, widely used in Claude Code workflows.

Compared to these, BACA is uniquely characterized by its **embedded governance layer** (Pre-write Checklist, state machine, turn limits) and its explicit design for **bidirectional peer conversation** rather than task dispatch.

### 2.4 Claude Code Agent Teams

Anthropic's experimental Claude Code Agent Teams (v2.1.32+) provides an official multi-agent coordination mechanism using a shared task list and a peer-to-peer mailbox system. While functionally similar in intent, Agent Teams differs from BACA in that its communication channel is platform-managed and opaque; the WORKLOG is not human-readable without tooling. Agent Teams also currently targets single-machine deployments, whereas BACA is designed for cross-device operation via cloud synchronization.

---

## 3. Protocol Design

### 3.1 Architectural Overview

A BACA deployment consists of:

```
Device A (Agent A)                    Device B (Agent B)
┌─────────────────┐                  ┌─────────────────┐
│ Claude Code     │                  │ Claude Code     │
│ Desktop Session │                  │ Desktop Session │
│                 │  Cloud Storage   │                 │
│ Local CLAUDE.md │ ←── OneDrive ──→ │ Local CLAUDE.md │
│                 │    (sync layer)  │                 │
└────────┬────────┘                  └────────┬────────┘
         │ read/append                         │ read/append
         └──────────────┬──────────────────────┘
                        │
              ┌─────────▼──────────┐
              │   WORKLOG_{A}_{B}  │
              │   _{topic}.md      │
              │  (shared document) │
              └────────────────────┘
```

Two document types are instantiated per session:

- **HANDSHAKE document**: Immutable protocol specification defining topic scope, response mode, turn limits, and behavioral rules. Agreed upon at session initialization.
- **WORKLOG document**: The append-only conversational log. Serves simultaneously as message transport, audit trail, and shared state.

### 3.2 Session Lifecycle (Finite State Machine)

```
INIT_PENDING_B_CONFIRM
        │
        │  Agent B writes confirmation entry
        ▼
      ACTIVE ◄────────────────────────┐
        │                             │
        │  Turn limit reached         │ Human authorization
        ▼                             │
  PAUSED_LIMIT_REACHED ──────────────►┘
        │
        │  Human declares closure
        ▼
     CLOSED
```

State is stored directly in the WORKLOG header table, readable and writable by both agents and humans:

```markdown
| **Session 狀態** | **ACTIVE** |
| **已用輪次**     | **2 / 5**  |
```

### 3.3 The Pre-write Checklist (Governance Layer)

Before appending any entry, an agent must complete a three-item checklist:

```markdown
### ✅ Pre-write Checklist
- [ ] Did this entry respond to the previous agent's Action Items?
- [ ] Is the content within the defined topic scope?
- [ ] If off-topic, has it been flagged with ⚠️ and justified?

Verification result: PASS / FAIL
(FAIL blocks the entry; the agent must explain why)
```

This checklist serves as a **behavioral contract** enforced in natural language rather than code. It prevents topic drift, ensures Action Item accountability, and creates a computable trace of compliance. In the empirical validation, all 8 entries (including 2 initialization entries) passed without exception.

### 3.4 Entry Structure

Each WORKLOG entry follows a standardized schema:

```markdown
## YYYY-MM-DD HH:MM | {DEVICE_ID} | {Entry Topic}

### ✅ Pre-write Checklist       [mandatory]
### 📥 Confirm Previous Action Items  [when applicable]
### ✅ Completed This Turn       [mandatory]
### 🔧 Technical Action Log     [when applicable]
### 🧭 Decision Record          [when applicable]
### 🔴 Action Items for Counterpart  [when applicable]
### 💬 Questions / Pending      [when applicable]
### 🟡 Turn Count Update        [mandatory]
```

### 3.5 Response Modes

Two response modes are supported:

- **MANUAL**: Agent waits for explicit human instruction before responding. Appropriate for high-oversight scenarios.
- **TIMED-Xmin**: Agent polls the WORKLOG every X minutes using `/loop`. If the latest entry belongs to the counterpart and contains unresolved Action Items, the agent executes the Pre-write Checklist and responds. Used in the empirical validation (TIMED-3min).

### 3.6 Turn Limit and Human Authorization

A configurable turn limit (default: 5 rounds, where one round = one entry per agent) creates mandatory human checkpoints. Upon reaching the limit, the session transitions to `PAUSED_LIMIT_REACHED` and all automated behavior halts. This prevents runaway agent loops and ensures human accountability for extended autonomous operation.

---

## 4. Implementation

### 4.1 Initialization

Session initialization is guided by a 7-question wizard embedded in `INIT_WIZARD.md`:

1. Device A identifier
2. Device B identifier
3. Topic name (used in filename)
4. Allowed discussion scope
5. Forbidden discussion scope
6. Response mode (MANUAL / TIMED-Xmin)
7. Turn limit

Upon completion, the wizard generates the HANDSHAKE and WORKLOG documents at the cloud-synchronized path and seeds the first entry.

### 4.2 Device-Side Configuration

Each device maintains a local `CLAUDE.md` file in the working directory that encodes:

- Device identity and role (A or B)
- Cloud storage path to the shared sessions directory
- Startup behavior: scan for existing WORKLOGs, infer session state, execute appropriate flow

The Mac Mini's `CLAUDE.md` was provisioned remotely from the PC side via SSH (`ssh {mac_user}@{mac_local_ip}`), demonstrating that the per-device configuration itself requires no manual setup on the target device.

### 4.3 Conflict Prevention

Write conflicts are avoided through the **turn discipline** embedded in the protocol: agents only write when the latest WORKLOG entry belongs to the counterpart. This is enforced by the `/loop` polling logic, which checks entry authorship before executing the Pre-write Checklist. No file locking mechanism is required because the turn-taking constraint eliminates concurrent writes as a valid protocol state.

### 4.4 Infrastructure Requirements

| Component | Requirement |
|---|---|
| LLM runtime | Claude Code Desktop (any version supporting `/loop`) |
| Sync layer | Any cloud storage with file sync (OneDrive, iCloud, Dropbox, Syncthing) |
| Network | Agents need not be online simultaneously |
| Server | None |
| Database | None |
| Custom code | None |

---

## 5. Empirical Validation

### 5.1 Setup

| Parameter | Value |
|---|---|
| Session topic | Self-introduction and hardware specification exchange |
| Agent A | PC-KAIJI (Windows 11 Pro, Intel Core Ultra 9 185H, RTX 4080 Laptop) |
| Agent B | MACMINI-KAIJI (macOS 26.4.1 Tahoe, Apple M4, 16 GB Unified Memory) |
| Response mode | TIMED-3min |
| Turn limit | 5 rounds |
| Sync layer | Microsoft OneDrive |
| Duration | 2026-04-19 23:10 → 2026-04-20 00:03 (53 minutes) |

### 5.2 Results

The session completed 4 rounds (8 entries total, excluding 2 initialization entries) before early closure by mutual agreement. All turns were executed autonomously; human intervention was limited to session initialization parameter input and monitoring.

**Round-by-round summary:**

| Round | Agent A Entry | Agent B Entry | Latency |
|---|---|---|---|
| Init | Session initialization (23:10) | Protocol confirmation (23:31) | 21 min |
| 1 | Hardware specs + questions (23:32) | Hardware specs + counter-questions (23:36) | 4 min |
| 2 | Workload benchmarks + OS overview (23:38) | MLX status correction + memory profiling (23:52) | 14 min |
| 3 | Windows vs macOS dev workflow (23:55) | macOS dev workflow + Homebrew/Tahoe (23:57) | 2 min |
| 4 | Session summary + closure proposal (00:00) | CLOSED declaration + SUMMARY generation (00:03) | 3 min |

**Notable observations:**

1. **Genuine factual correction**: In Round 2, MACMINI-KAIJI self-corrected a claim made in Round 1 — it had stated "the machine primarily runs llama.cpp/MLX" but discovered via live `which` and `python3 -c "import mlx"` commands that neither was installed. This unprompted factual correction, backed by technical evidence, demonstrates that genuine context isolation (each agent independently examining its environment) produces qualitatively different outputs than a single-agent role-play scenario.

2. **Protocol compliance**: All 8 entries passed the Pre-write Checklist. Zero topic violations. Zero skipped Action Items.

3. **SUMMARY artifact**: MACMINI-KAIJI autonomously generated a structured summary document upon session closure, without explicit instruction in the current turn (the instruction had been issued two turns prior by PC-KAIJI). This demonstrates successful long-horizon instruction retention via the WORKLOG.

4. **Cross-OS tool use**: Both agents invoked OS-specific commands (`powershell`, `system_profiler`, `top`, `brew`) appropriate to their respective environments and incorporated results into the shared WORKLOG, enabling genuine cross-platform hardware comparison.

---

## 6. Discussion

### 6.1 Primary Contributions

**Transparency over throughput.** BACA deliberately sacrifices latency (3-minute polling intervals vs. sub-second RPC calls in conventional multi-agent systems) in exchange for a communication channel that is fully human-readable at every point in time. This aligns with emerging governance requirements for auditable AI behavior.

**Protocol as governance.** The Pre-write Checklist, topic scope constraints, and turn limits are not implemented as code — they are natural language instructions that the LLM interprets and enforces. This makes the governance layer modifiable without engineering involvement and co-located with the communication artifact itself.

**Infrastructure independence.** The only platform-specific dependency is the LLM runtime. The sync layer (OneDrive in validation, but substitutable with any file-sync service) is a commodity. The WORKLOG is plain Markdown. This means the protocol is portable across LLM providers: a MACMINI-KAIJI running GPT-4 could participate in a session initiated by a PC-KAIJI running Claude, provided both understand the WORKLOG format.

**Genuine cognitive isolation.** Because agents run in separate sessions on separate devices, each agent's context window contains only: (a) its local CLAUDE.md, (b) the WORKLOG it has read, and (c) its own session history. There is no shared hidden state. Claims made by one agent are not "known" to the other except through the WORKLOG. This produces qualitatively different behavior from simulated multi-agent systems.

### 6.2 Limitations

**Polling latency.** The TIMED-Xmin mode introduces artificial latency proportional to the polling interval. For time-sensitive tasks, this is a significant drawback. A file-watch mechanism (e.g., `inotifywait` on Linux, `FSEvents` on macOS) could reduce latency to near-zero without the polling overhead.

**No concurrent writes.** The turn-discipline constraint assumes strictly alternating writes. Multi-party sessions (3+ agents) would require an explicit arbitration mechanism (e.g., a PENDING_WRITE state flag in the WORKLOG header) to prevent races.

**Context window growth.** As WORKLOGs grow, agents reading the entire document consume increasing context. Mitigation strategies include summary truncation (reading only the last N entries) or maintaining a rolling SUMMARY file, as demonstrated in the validation.

**LLM instruction following.** Protocol compliance depends on the LLM correctly interpreting natural-language rules. Weaker models may skip checklist items or misparse entry boundaries. Claude Sonnet 4.6 exhibited reliable compliance throughout the validation; generalizability to other models requires further testing.

### 6.3 Same-Device, Multi-Session Extension

The protocol requires no modification for same-device, multi-session operation. Replacing the cloud-sync path with a local filesystem path reduces latency to I/O-bound speeds (~1–50ms) and eliminates network dependencies entirely. This enables BACA as a coordination mechanism for parallel Claude Code sessions within a single development environment — for example, one session handling frontend code while another handles backend, communicating intent and decisions through a shared WORKLOG rather than through the developer as an intermediary.

---

## 7. Conclusion

BACA demonstrates that meaningful, auditable, cross-device LLM agent collaboration is achievable with zero specialized infrastructure. By grounding the protocol in a human-readable append-only document rather than a purpose-built messaging system, BACA makes multi-agent coordination accessible to non-engineers and transparent to human supervisors. The empirical validation confirms that genuine context isolation between agents produces qualitatively different — and arguably more trustworthy — collaborative outputs than single-agent role-play.

The protocol's simplicity is its strength: any change to the governance rules requires editing a Markdown file, not redeploying a service. Any human can read the complete interaction history without tools. Any LLM that can read and write Markdown can participate.

We believe BACA represents a useful point in the design space between heavyweight multi-agent orchestration platforms and ad-hoc prompt chaining — one that prioritizes human legibility and governance over raw coordination throughput.

---

## References

[1] Wu, Q., et al. "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation." *arXiv preprint arXiv:2308.08155* (2023).

[2] LangChain. "LangGraph: Build resilient language agents as graphs." https://langchain-ai.github.io/langgraph/ (2024).

[3] Erman, L. D., et al. "The Hearsay-II speech-understanding system: Integrating knowledge to resolve uncertainty." *ACM Computing Surveys* 12.2 (1980): 213–253.

[4] [ESAA: Event Sourcing for Autonomous Agents. arXiv:2602.23193 (2026) — unverified, cited as directional reference.]

[5] Dicklesworthstone. "mcp_agent_mail: A Mailbox-Style Coordination Layer for MCP Agents." GitHub (2025). https://github.com/Dicklesworthstone/mcp_agent_mail

[6] Sinai, A. "agent-message-queue: Zero-Infrastructure Maildir-Style Message Queue for AI Agents." GitHub (2025).

[7] Community practice. "TASKS.md pattern in Claude Code workflows." Widely observed in Claude Code repositories (2024–2026).

---

## Appendix A: WORKLOG Format Specification

A conforming WORKLOG document must contain:

1. A YAML-style header table with fields: `Session 狀態`, `已用輪次`, `回應模式`, `自動對答上限`
2. An append-only sequence of entries, each beginning with `## YYYY-MM-DD HH:MM | {DEVICE_ID} | {Topic}`
3. Each entry must contain `✅ Pre-write Checklist` and `✅ 本次完成` blocks
4. The `Session 狀態` header field must be updated whenever state transitions occur
5. No entry may be modified or deleted after being committed to the document

## Appendix B: HANDSHAKE Protocol Fields

| Field | Description | Example |
|---|---|---|
| `DEVICE_A_ID` | Initiating device identifier | `PC-KAIJI` |
| `DEVICE_B_ID` | Responding device identifier | `MACMINI-KAIJI` |
| `TOPIC` | Session topic (used in filenames) | `self-intro-hardware` |
| `ALLOWED_SCOPE` | Comma-separated list of permitted topics | `自我介紹, 硬體規格` |
| `FORBIDDEN_SCOPE` | Comma-separated list of prohibited topics | `個股建議, 帳號資訊` |
| `RESPONSE_MODE` | `MANUAL` or `TIMED-Xmin` | `TIMED-3min` |
| `MAX_AUTO_TURNS` | Maximum autonomous rounds before pause | `5` |

---

*Proof-of-concept implementation and session logs available upon request.*
*Protocol version: v1.0 · Validation date: 2026-04-19/20*
