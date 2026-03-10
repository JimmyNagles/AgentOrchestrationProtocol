# Agent Orchestration Protocol v2.1

An operating system for solo founders who want to move fast using multiple AI agents — without chaos.

You act as CEO. The agents execute. This protocol defines who does what, how they communicate, and when code actually gets written.

---

## The Problem This Solves

Running multiple AI agents in parallel is powerful but messy. Without structure you get:
- Agents overwriting each other's work
- Code written before the idea is clear
- No record of what was done or why
- The founder spending more time managing agents than building

This protocol fixes that with clear roles, async communication, and a gating system that prevents wasted effort.

---

## How It Works

### The Team

| Agent | Tool | Role |
|-------|------|------|
| **Cowork** | Claude (UI/Projects) | Chief of Staff — coordinates everything, writes specs, drains agent OUTBOXes |
| **Claude Code** | Claude Code (terminal) | Backend Domain A — DB schema, migrations, API routes |
| **Codex** | Codex (terminal) | Backend Domain B — business logic, intelligence layer |
| **Gemini** | Gemini / Antigravity | Frontend — all pages, components, UI + QA |
| **You** | Your brain | CEO — make decisions, review work, push to GitHub |

Agents never talk to each other directly. All coordination flows through you and Cowork.

### Phase 0: Ideation (Before Any Code)

The `ideation/` folder contains 6 templates you fill in before a single line of code is written:

1. **PROBLEM.md** — What problem are you solving and for whom?
2. **USER.md** — Who exactly is the target user?
3. **SOLUTION.md** — What are you building and what's out of scope?
4. **MARKET.md** — Who are the competitors and how do you win?
5. **VALIDATION.md** — What assumptions need to be tested?
6. **HERO-MOMENT.md** — The exact moment a user realizes this product is for them

**No agent writes code until `HERO-MOMENT.md` is defined.** This is the most important gate in the protocol.

Cowork walks you through all of these with guided questions. You don't fill them in alone.

### Phase 1+: Building

Once ideation is complete, Cowork plans the work and assigns tasks to each agent. The cycle looks like this:

```
You → Cowork: "check status"
Cowork: reads all agent OUTBOXes → publishes to AGENT-STATUS.md → clears OUTBOXes → writes next tasks
You: send cold-start messages to each agent with their new task
Agents: execute → write completion to their OUTBOX
(repeat)
```

Agents work in parallel on separate domains to minimize merge conflicts.

### Implementation Order

```
DB Schema (Claude Code)
    ↓
API Routes (Claude Code)
    ↓
Business Logic (Codex)
    ↓
Frontend UI (Gemini)
    ↓
QA Testing (Gemini)
    ↓
You review → push to GitHub
```

---

## File Structure

```
/
├── AGENT-PROTOCOL.md       # Core rules every agent must follow
├── AGENT-STATUS.md         # Real-time team dashboard (Cowork manages this)
├── ROADMAP.md              # Product vision, phases, codebase map
├── SHARED.md               # Shared knowledge: APIs, DB schema, test accounts, bugs
│
├── ideation/
│   ├── PROBLEM.md
│   ├── USER.md
│   ├── SOLUTION.md
│   ├── MARKET.md
│   ├── VALIDATION.md
│   └── HERO-MOMENT.md      # The gate. Nothing ships without this.
│
├── agents/
│   ├── _template.md        # Blueprint for adding new agents
│   ├── cowork.md           # Cowork's current task + completed log + OUTBOX
│   ├── claude-code.md      # Claude Code's current task + log + OUTBOX
│   ├── codex.md            # Codex's current task + log + OUTBOX
│   └── gemini.md           # Gemini's current task + log + OUTBOX
│
└── designs/                # Design screenshots by phase
```

---

## Getting Started

### 1. Clone and initialize

```bash
git clone git@github.com:JimmyNagles/AgentOrchestrationProtocol.git my-project
cd my-project
```

Search and replace every placeholder in the files:
- `[PROJECT NAME]` → your project name
- `[branch]` → your branch naming convention
- `[tech stack]` → your stack (e.g. Next.js, Supabase, etc.)

Clear the agent log sections (keep the headers, remove example entries).

### 2. Start Cowork

Open a new Claude Project and paste this cold-start message:

```
You are Cowork, Chief of Staff for [PROJECT NAME].

Please read these files in order:
1. AGENT-PROTOCOL.md
2. ideation/PROBLEM.md
3. ideation/USER.md
4. ideation/SOLUTION.md
5. ideation/MARKET.md
6. ideation/VALIDATION.md
7. ideation/HERO-MOMENT.md
8. AGENT-STATUS.md
9. agents/cowork.md
10. agents/claude-code.md
11. agents/codex.md
12. agents/gemini.md
13. ROADMAP.md
14. SHARED.md

Once you've read everything, check the ideation/ files. If any are incomplete, walk me through them using the Agent Guide questions in each file. Do not move to planning until HERO-MOMENT.md is filled in.
```

### 3. Complete ideation

Cowork will ask you questions. Answer them. It will fill in the templates as you go.

### 4. Assign and execute

Once ideation is done, tell Cowork: `"check status"` — it will write tasks for each agent and update AGENT-STATUS.md.

Then open each agent in its respective tool (Claude Code terminal, Codex terminal, Gemini) and send the cold-start message from `AGENT-PROTOCOL.md`.

### 5. Adding a new agent

Copy `agents/_template.md`, rename it, and update it with the agent's role, branch, and domain. Then add it to the roster in `AGENT-PROTOCOL.md`.

---

## Core Rules (For Agents and Founders)

- **One task at a time.** Finish before starting new.
- **Stay in your lane.** Don't do another agent's job.
- **Use your OUTBOX.** Never edit AGENT-STATUS.md directly — that's Cowork's job.
- **Never push to GitHub.** Only the founder pushes.
- **Never push to main.** Always use a feature branch.
- **When in doubt, stop and ask.** Don't guess.
- **No UI changes without founder approval.**

---

## Origin

This protocol was proven while building ConciergeAI and refined into a reusable template. The core insight: agents need the same clarity a human team needs — defined roles, async communication, and a clear definition of done.

---

## License

MIT — use it, fork it, adapt it for your stack.
