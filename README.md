# Agent Protocol v4.0

An operating system for solo founders who orchestrate AI agent teams.

You act as CEO. The orchestrator conducts. The agents execute. This protocol defines who does what, how they communicate, and how quality is maintained — inspired by distributed systems, biological swarms, NASA mission control, and kitchen brigades.

---

## The Problem

Running multiple AI agents is powerful but chaotic. Without structure:
- Agents overwrite each other's work
- Failed agents go undetected
- Context is lost between sessions
- Nobody knows what's done, what's blocked, or what's next
- One bad output can cascade through the whole system

This protocol fixes that with battle-tested coordination patterns stolen from systems that solve the same problems at scale.

---

## The Core Idea

**AI agents are stateless. Files are not.**

Every agent gets its own `.md` file. That file carries the agent's current task, status, completed work, and outbox messages across sessions. When a new session starts, the agent reads its file and picks up where it left off.

Agents communicate through shared files — like ants leaving pheromone trails. The environment IS the communication medium.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        FOUNDER (CEO)                         │
│              Approves direction. Reviews work.               │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   ORCHESTRATOR (Conductor)                    │
│     Main AI session. Writes tasks. Launches agents.          │
│     Monitors outboxes. Coordinates handoffs.                 │
│     Runs consensus checks. Never builds — only conducts.     │
└───┬──────────┬──────────┬──────────┬──────────┬─────────────┘
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
┌────────┐┌────────┐┌────────┐┌────────┐┌────────┐
│Agent A ││Agent B ││Agent C ││Agent D ││Agent E │
│  .md   ││  .md   ││  .md   ││  .md   ││  .md   │
└───┬────┘└───┬────┘└───┬────┘└───┬────┘└───┬────┘
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
┌─────────────────────────────────────────────────────────────┐
│                    SHARED STATE (Files)                       │
│  SHARED.md │ AGENT-STATUS.md │ ROADMAP.md │ Code files       │
│                                                               │
│  Agents communicate through the environment, not directly.   │
│  Like ants leaving pheromone trails — stigmergy.             │
└─────────────────────────────────────────────────────────────┘
```

**The Founder** is the CEO — approves direction, reviews work, pushes to GitHub.

**The Orchestrator** is the conductor — breaks down work, writes tasks to agent files, launches agents, monitors progress, coordinates handoffs, runs consensus checks. The orchestrator never builds — only conducts.

**Agents** are the workers — each owns a domain (a set of files/directories), reads their task from their `.md` file, executes, and reports back via their OUTBOX.

**Shared State** is the communication medium — agents read and write to shared files. No direct agent-to-agent messaging. The files ARE the message bus.

---

## What Makes v4.0 Different

Six improvements inspired by real-world coordination systems, validated through multi-agent consensus (4 agents independently proposed the same patterns from different domains):

### 1. Closed-Loop Communication

*Inspired by: aviation read-backs, ER team protocols*

Agents don't just read their task and go. They confirm what they understood:

```
Orchestrator writes task → Agent reads it → Agent writes read-back:
"I will [specific action]. Files: [list]. Output: [expected result]."
→ Orchestrator verifies → Agent proceeds
```

Prevents "I thought you meant..." errors.

### 2. Pre-Flight Checklist

*Inspired by: aviation pre-flight, kitchen mise en place*

Every agent, every task, same checklist before starting:

```
[x] Read AGENT-PROTOCOL.md
[x] Read SHARED.md
[x] Read AGENT-STATUS.md
[x] Read my agent file
[x] Confirmed dependencies are met
[x] Confirmed files are in my domain
```

Agents that skip context produce garbage. This ensures a consistent baseline.

### 3. Task Lifecycle States

*Inspired by: Kanban, CI/CD pipelines*

Not just "Waiting" and "Done" — explicit states with clear meaning:

```
QUEUED → CLAIMED → IN_PROGRESS → REVIEW → DONE

Special states: BLOCKED (can't proceed) | FAILED (tried, didn't work)
```

### 4. Severity Levels

*Inspired by: incident response, military rules of engagement*

Not all tasks need the same process:

| Severity | Autonomy | Example |
|----------|----------|---------|
| LOW | Agent executes freely | Fix a typo |
| MEDIUM | Agent executes, orchestrator reviews | Add an API endpoint |
| HIGH | Orchestrator reviews plan first | Change DB schema |
| CRITICAL | Consensus mode + Go/No-Go poll | Deploy to production |

### 5. Circuit Breaker

*Inspired by: distributed systems circuit breakers*

If an agent fails twice on the same task type, stop sending it work:

```
CLOSED (normal) → 2 failures → OPEN (stopped)
                                    │
                               Fix the issue
                                    │
                                    ▼
                              HALF-OPEN (test)
                                    │
                              ┌─────┴─────┐
                           Pass          Fail
                              │             │
                         CLOSED          OPEN (escalate)
```

### 6. Consensus Mode

*Inspired by: Byzantine fault tolerance, peer review, newsroom fact-checking*

For high-stakes decisions, send the same task to 3+ agents independently:

```
Same task → Agent A findings
         → Agent B findings    →  Compare:
         → Agent C findings       3/3 agree = accept
                                  2/3 agree = investigate
                                  1/3 only  = verify or discard
```

No single agent is trustworthy. The consensus of multiple independent agents is high-confidence.

---

## The Task Flow

```
You: "Build me an auth system"
          │
          ▼
Orchestrator breaks it into tasks:
  ├── Backend: "Build auth API endpoints"     (MEDIUM)
  ├── Frontend: "Build login/signup pages"    (MEDIUM)
  ├── Designer: "Style the auth flow"         (LOW)
  └── Security audit before deploy            (CRITICAL — consensus mode)
          │
          ▼
Orchestrator writes tasks to each agent's .md file
          │
          ▼
Each agent completes pre-flight checklist
Each agent writes read-back confirmation
          │
          ▼
Orchestrator verifies read-backs, launches agents:
  claude -p "cold-start message" --dangerously-skip-permissions
          │
          ▼
Agents work in parallel (different domains, no file overlap)
Agents update status: CLAIMED → IN_PROGRESS → REVIEW
          │
          ▼
Agents finish, write results to OUTBOX
          │
          ▼
Orchestrator reads outboxes:
  ├── Work looks good → DONE, assign next task
  ├── Work has issues → feedback to agent, stays IN_PROGRESS
  ├── Agent failed → circuit breaker, reassign
  └── High-stakes → consensus mode before accepting
          │
          ▼
You review and push to GitHub
```

---

## File Structure

```
project/
├── AGENT-PROTOCOL.md        # This file — the rules
├── AGENT-STATUS.md          # Live dashboard (orchestrator manages)
├── ROADMAP.md               # Product vision and phases
├── SHARED.md                # Shared knowledge: APIs, bugs, architecture
│
├── ideation/                # Phase 0 — define before building
│   ├── PROBLEM.md
│   ├── USER.md
│   ├── SOLUTION.md
│   ├── MARKET.md
│   ├── VALIDATION.md
│   └── HERO-MOMENT.md       # The gate — nothing ships without this
│
├── agents/                  # One file per agent — this IS their state
│   ├── _template.md         # Blueprint for new agents
│   ├── backend.md
│   ├── frontend.md
│   ├── designer.md
│   ├── research.md
│   └── product.md
│
└── designs/                 # Design references by phase
```

---

## Getting Started

### 1. Clone

```bash
git clone git@github.com:JimmyNagles/AgentOrchestrationProtocol.git my-project
cd my-project
```

### 2. Set up agents

For each agent you need, copy `agents/_template.md` and name it by role:
- `agents/backend.md`
- `agents/frontend.md`
- `agents/research.md`

Define their DOMAIN OWNERSHIP (which files they can touch).

### 3. Start orchestrating

Open Claude Code. You are the orchestrator. Write tasks to agent files, launch them:

```bash
claude -p "You are BACKEND on [project].
1. Read AGENT-PROTOCOL.md
2. Read agents/backend.md for your task
3. Read SHARED.md
Complete pre-flight. Write read-back. Execute. Update OUTBOX." \
--dangerously-skip-permissions
```

### 4. Monitor and iterate

Read agent outboxes. Verify work. Assign next tasks. Repeat.

---

## Patterns We Stole From

| Pattern | Source | How We Use It |
|---------|--------|---------------|
| Stigmergy | Ant colonies | Agents communicate through shared files, not direct messages |
| Quorum sensing | Bee swarms | Consensus mode — majority agreement = confidence |
| Circuit breaker | Distributed systems | Stop sending work to failing agents |
| Read-back | Aviation / ER teams | Agents confirm understanding before executing |
| Go/No-Go polling | NASA mission control | Single NO-GO halts irreversible actions |
| Mise en place | Kitchen brigade | Pre-flight checklist before every task |
| MAINTAINERS file | Linux kernel | Domain ownership — each file has one owner |
| CODEOWNERS | GitHub | File-path patterns route work to the right agent |
| Byzantine fault tolerance | Blockchain / Paxos | Multiple independent agents, same task, compare results |
| Incident command | ER / Fire teams | Clear roles, clear authority, clear escalation |
| Brigade de cuisine | Restaurant kitchens | Specialized stations, one expediter, hands-off leader |

---

## Origin

This protocol was invented while building ConciergeAI, refined across multiple projects, and stress-tested through multi-agent consensus research. The v4.0 improvements were proposed by 4 independent AI agents analyzing coordination patterns from distributed systems, biology, software engineering, and human organizations — and validated by finding which patterns 3+ agents independently agreed on.

The core insight: **agents need the same clarity the best human teams need — defined roles, async communication, quality gates, and a clear definition of done.**

---

## License

MIT — use it, fork it, adapt it.
