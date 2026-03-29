# AGENT PROTOCOL v4.0
> The operating system for solo founder + AI agent teams.
> Inspired by distributed systems, biological swarms, NASA mission control, and kitchen brigades.
> One agent. One file. One domain. That's the foundation.

---

## What Is This?

This protocol turns a solo founder into a one-person startup by coordinating a team of AI agents — each with a defined role, clear responsibilities, and rules for working together.

**The founder is always the CEO.** The orchestrator (main AI session) is the conductor. Agents execute. Nobody makes strategic decisions without the founder's approval.

### Why Files = State

AI agents are stateless. Every session starts fresh. The `.md` file for each agent IS that agent's memory — it carries the current task, completed work, and outbox messages across sessions. **Every agent gets its own file. No exceptions.**

### Architecture

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
│ file   ││ file   ││ file   ││ file   ││ file   │
└───┬────┘└───┬────┘└───┬────┘└───┬────┘└───┬────┘
    │          │          │          │          │
    ▼          ▼          ▼          ▼          ▼
┌─────────────────────────────────────────────────────────────┐
│                    SHARED STATE (Files)                       │
│  SHARED.md │ AGENT-STATUS.md │ ROADMAP.md │ Code files       │
│                                                               │
│  Agents read/write to shared state — this IS the              │
│  communication medium. Like ants leaving pheromone trails.    │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Rules (All Agents Must Follow)

### The 12 Rules

1. **Pre-flight before acting.** Complete the Pre-Flight Checklist before starting any task.
2. **Read-back your task.** After reading your task, write a 1-2 sentence confirmation of what you're about to do in your agent file under CURRENT TASK. The orchestrator verifies before you proceed.
3. **Stay in your domain.** Only modify files listed in your DOMAIN OWNERSHIP section. If you need a file outside your domain, report it as a blocker.
4. **Update your status honestly.** Use the Task Lifecycle states: QUEUED → CLAIMED → IN_PROGRESS → REVIEW → DONE. Update as you transition.
5. **Use your OUTBOX.** Write completion messages to your OUTBOX. Only the orchestrator edits AGENT-STATUS.md.
6. **Never push to main.** Always push to a feature branch. Founder merges. No exceptions.
7. **Never assign tasks to other agents.** Only the founder or orchestrator assigns tasks.
8. **One task at a time.** Finish what you're doing before starting something new.
9. **When in doubt, stop and ask.** Don't guess on important decisions. Write your question to your OUTBOX and set status to BLOCKED.
10. **Verify your own work.** After completing a task, test what you created. Include proof in your OUTBOX (build output, test results, curl response).
11. **Report blockers, don't improvise.** If something is broken or unavailable, report it. Don't silently work around it.
12. **Go/No-Go on irreversible actions.** Never deploy, delete, or make architectural changes without orchestrator approval. Write your plan to OUTBOX first.

---

## Pre-Flight Checklist

> Every agent, every task, every time. No exceptions.

Before starting any work, complete this checklist and note it in your agent file:

```
PRE-FLIGHT:
[x] Read AGENT-PROTOCOL.md — understood the rules
[x] Read SHARED.md — checked architecture, API contracts, known bugs
[x] Read AGENT-STATUS.md — checked team status, blockers, recent decisions
[x] Read my agent file — understood my task, domain, and history
[x] Confirmed dependencies — any upstream work I depend on is DONE
[x] Confirmed domain — files I'll touch are in my DOMAIN OWNERSHIP
```

If any item fails, set status to BLOCKED and explain why in your OUTBOX.

**Why this exists:** Agents that skip context produce garbage. This checklist ensures every agent starts from the same baseline. Inspired by aviation pre-flight checks and kitchen mise en place — preparation prevents errors.

---

## Closed-Loop Communication (Read-Back Rule)

> Inspired by aviation and ER teams. Prevents "I thought you meant..." errors.

### How it works

```
Orchestrator writes task to agent file
        │
Agent reads task
        │
Agent writes read-back: "I understand my task is to [X].
I will modify [files]. I will NOT touch [files].
Expected output: [Y]."
        │
Orchestrator verifies read-back
        │
        ├── Correct → Agent proceeds
        │
        └── Wrong → Orchestrator clarifies, agent re-reads
```

### Read-Back Format

In the CURRENT TASK section of your agent file, after reading the task, add:

```
## CURRENT TASK
- Task: [assigned task]
- Branch: [branch]
- Status: CLAIMED
- Read-back: I will [specific action]. Files: [list]. Output: [expected result].
```

The orchestrator checks this before the agent is launched. If the read-back is wrong, the task is clarified before any work happens.

---

## Task Lifecycle

> Every task moves through explicit states. No ambiguity about where things stand.

```
QUEUED → CLAIMED → IN_PROGRESS → REVIEW → DONE
  │         │          │            │        │
  │         │          │            │        └── Work accepted, outbox drained
  │         │          │            └── Work complete, waiting for verification
  │         │          └── Agent is actively working
  │         └── Agent has read and confirmed the task (read-back done)
  └── Task written to agent file, agent not yet started
```

### Special States

```
BLOCKED  — Cannot proceed. Dependency not met, question unanswered, or error encountered.
FAILED   — Agent attempted but could not complete. Requires reassignment or investigation.
```

### Rules

- Tasks can only move forward (QUEUED → CLAIMED → IN_PROGRESS → REVIEW → DONE)
- Except: any state can transition to BLOCKED or FAILED
- BLOCKED tasks must include a reason in the OUTBOX
- FAILED tasks must include what was tried and why it failed in the OUTBOX
- Only the orchestrator moves tasks from REVIEW → DONE (verification required)

---

## Task Severity Levels

> Not all tasks need the same process. Severity determines autonomy.

| Severity | Description | Agent Autonomy | Example |
|----------|-------------|----------------|---------|
| **LOW** | Small, safe, reversible changes | Agent executes freely. Orchestrator reviews after. | Fix a typo, add a log line, update a comment |
| **MEDIUM** | Standard feature work within domain | Agent executes. Orchestrator reviews output. | Add an API endpoint, build a component, write tests |
| **HIGH** | Cross-domain, architectural, or risky changes | Orchestrator reviews plan BEFORE agent executes. | Change DB schema, modify auth flow, refactor shared code |
| **CRITICAL** | Irreversible, security-sensitive, or high-stakes | Consensus mode — multiple agents verify. Go/No-Go poll before execution. | Deploy to production, delete data, change payment logic |

### Default

If no severity is specified, assume **MEDIUM**.

### The Go/No-Go Rule (CRITICAL tasks only)

Before any CRITICAL task executes:
1. Orchestrator describes the action and its consequences
2. Orchestrator polls available agents: "GO or NO-GO?"
3. Each agent responds with unambiguous GO or NO-GO plus a one-line reason
4. **Single NO-GO halts everything.** No "maybe." No "probably fine."
5. The NO-GO agent must explain the concern. Orchestrator resolves before re-polling.

Inspired by NASA Flight Director polling before irreversible mission events.

---

## Circuit Breaker

> If an agent keeps failing, stop sending it work. Inspired by distributed systems circuit breakers.

### Three States

```
CLOSED (normal) → Agent receives tasks and executes normally
        │
  2 consecutive failures on same task type
        │
        ▼
OPEN (tripped) → Agent does NOT receive new tasks
        │         Orchestrator investigates: wrong prompt? missing context?
        │         capability mismatch? rate limit?
        │
  Orchestrator fixes the issue
        │
        ▼
HALF-OPEN (testing) → Agent gets ONE simple task to verify the fix
        │
        ├── Success → Back to CLOSED
        │
        └── Failure → Back to OPEN, escalate to founder
```

### What counts as a failure

- Agent marks task DONE but OUTBOX is empty (delivered nothing)
- Agent modifies files outside its DOMAIN OWNERSHIP
- Agent's work breaks the build or fails tests
- Agent doesn't update its status (silent failure)

### What the orchestrator does when a circuit opens

1. Log the failure pattern in AGENT-STATUS.md
2. Reassign the task to a different agent
3. Investigate: was the prompt unclear? Was context missing? Was the agent the wrong fit?
4. Fix the root cause before sending the agent another task

---

## Consensus Mode

> For high-stakes decisions, send the same task to multiple agents independently. Compare results. Accept what the majority agrees on. Inspired by Byzantine fault tolerance.

### When to use

- Security audits
- Architecture decisions
- Go/no-go evaluations
- Any task where being wrong is expensive

### How it works

```
Orchestrator writes the SAME task to 3+ agent files
        │
Each agent works INDEPENDENTLY (no reading other agents' work)
        │
All agents complete and write to OUTBOX
        │
Orchestrator compares all outputs
        │
        ├── Finding in 3+ reports → HIGH CONFIDENCE (accept)
        │
        ├── Finding in 2 reports → MODERATE CONFIDENCE (investigate)
        │
        └── Finding in 1 report → LOW CONFIDENCE (verify or discard)
```

### Rules

- Agents MUST NOT read each other's outboxes during consensus tasks
- Minimum 3 agents for consensus (2 is just disagreement, 3 breaks ties)
- The orchestrator synthesizes — agents don't vote directly
- Consensus results are logged in AGENT-STATUS.md DECISIONS LOG

### The 3f+1 Rule

To tolerate f unreliable agents, you need 3f+1 total. With 4 agents, you can tolerate 1 unreliable agent. With 7 agents, you can tolerate 2.

---

## Handoff Protocol

> When one agent's output is another agent's input. Inspired by kitchen expediting — the pass.

```
Producing agent finishes work
        │
Writes the CONTRACT to SHARED.md (API spec, schema, interface)
        │
Writes to OUTBOX: "Ready — contract in SHARED.md under [section]"
        │
Orchestrator verifies the contract is in SHARED.md
        │
Orchestrator writes the consuming agent's task, referencing the contract
        │
Consuming agent reads SHARED.md before starting
        │
        ├── Contract is clear → Proceed
        │
        └── Contract is missing or unclear → BLOCKED
```

Never assume another agent's output is ready. Check SHARED.md. If the contract isn't there, you're blocked — say so.

---

## Communication Protocol

> Agents communicate through shared state (files), not direct messages. Like ants leaving pheromone trails — the environment IS the communication medium.

### The Loop

```
Orchestrator writes tasks to agent files
        │
Agents execute, update their files (status + outbox)
        │
Orchestrator reads all agent files ("check status")
        │
Orchestrator drains outboxes → publishes to AGENT-STATUS.md
        │
Orchestrator clears outboxes
        │
Orchestrator writes next tasks
        │
(repeat)
```

### The "check status" Command

When the founder says "check status" to the orchestrator:
1. Read all `agents/*.md` files
2. Check for any BLOCKED or FAILED agents — report immediately
3. Check circuit breaker states — any agents in OPEN?
4. Archive old messages in AGENT-STATUS.md
5. Drain each OUTBOX and publish new messages
6. Clear each agent's OUTBOX
7. Write next tasks into agent files if needed
8. Return brief status: what was done, what's blocked, what's next

---

## Decisions Protocol

> Important decisions must survive across sessions. Inspired by flight rules — pre-written decisions made in calm conditions.

**When to log:**
- Architectural choices
- Scope changes
- Tech stack choices
- Consensus results
- Any choice that affects multiple agents

**Who logs:** The orchestrator writes to the DECISIONS LOG in AGENT-STATUS.md during "check status". Any agent can request a decision be logged via their OUTBOX.

---

## Recovery Protocol

### Agent produces broken code
1. Set circuit breaker to OPEN for that agent
2. The domain owner fixes it (same agent after investigation, or reassign)
3. Never assign a different agent to fix another agent's domain without updating DOMAIN OWNERSHIP

### Agent writes to wrong files
1. Founder reverts the change (`git checkout -- <file>`)
2. Log the violation in AGENT-STATUS.md
3. Re-run with clearer domain boundaries in the agent file

### Agent session loses context
1. Start a new session with the cold-start message
2. The agent re-reads its `.md` file — the state is in the file, not in the session
3. This is why everything important goes in files, not in chat

### Build is broken and nobody knows why
1. Founder runs `git log` to find the last working commit
2. Orchestrator assigns the agent whose domain covers the broken area
3. Task: "Build is broken. Last working commit: [hash]. Diagnose and fix."
4. Severity: HIGH — orchestrator reviews diagnosis before agent fixes

### Agent fails repeatedly (circuit breaker OPEN)
1. Orchestrator investigates root cause (bad prompt? missing context? wrong agent?)
2. Fix the issue
3. Send a simple test task (HALF-OPEN state)
4. If test passes, resume normal operation (CLOSED)
5. If test fails, escalate to founder

---

## Ideation Phase (Phase 0)

> Before any roadmap or code exists, the idea needs to be defined. The `ideation/` folder handles this.

### Ideation Templates (in order)

| File | Purpose | Gate |
|------|---------|------|
| `ideation/PROBLEM.md` | Define the problem, who has it, and why current solutions fail | Must be filled before SOLUTION |
| `ideation/USER.md` | Define the target user | Must be filled before SOLUTION |
| `ideation/SOLUTION.md` | Define what we're building, MVP features, what's out of scope | Must be filled before MARKET |
| `ideation/MARKET.md` | Competitive landscape, positioning, pricing | Must be filled before VALIDATION |
| `ideation/VALIDATION.md` | Key assumptions and how to test them | Must be filled before Phase 1 |
| `ideation/HERO-MOMENT.md` | The user journey + the single "wow" moment | Must be filled before Phase 1 |

### The Gate Rule

**No agent writes code until ALL ideation files are complete.** If the founder can't articulate what the product feels like in one moment, the idea needs more work.

### Ideation Lock

Once Phase 1 begins, ideation files are **locked**. Changes require founder approval and must be noted in the DECISIONS LOG.

---

## Cold-Start Messages

> Every agent session starts fresh. The cold-start message is the boot sequence.

**Any agent:**
```
You are [AGENT NAME] on [PROJECT NAME].
1. Read AGENT-PROTOCOL.md — the rules
2. Read agents/[name].md — your task
3. Read SHARED.md — shared knowledge
4. Read AGENT-STATUS.md — team status
Complete the Pre-Flight Checklist. Write your read-back.
Execute your task. Update your status through the lifecycle.
When done: write results to your OUTBOX.
```

---

## Agent Roster

> Every agent gets its own `.md` file. The protocol is tool-agnostic — use any AI tool.

### Customizing

- **One agent or ten** — scale to your project
- **Name by role** — `agents/backend.md`, `agents/research.md`, not by tool
- **Define domains** — each agent's DOMAIN OWNERSHIP section lists which files they own
- Copy `agents/_template.md` for new agents

---

## Launching Agents

> The orchestrator launches agents via CLI. Agents need autonomous file/terminal access.

```bash
claude -p "[cold-start message]" --dangerously-skip-permissions
```

### Parallel Launch

Multiple agents can run simultaneously on different tasks. Rules:
- Agents must be working in different DOMAINS (no file overlap)
- The orchestrator monitors all agent files for status changes
- If two agents need the same file, they run sequentially, not in parallel

### Consensus Launch

For consensus mode, launch 3+ agents with the SAME task but writing to DIFFERENT agent files:
```bash
claude -p "You are AGENT-A. [task]. Write to agents/agent-a.md" &
claude -p "You are AGENT-B. [task]. Write to agents/agent-b.md" &
claude -p "You are AGENT-C. [task]. Write to agents/agent-c.md" &
wait
# Orchestrator compares all three outboxes
```

---

## Housekeeping

### Agent file hygiene
- **Current task** section: only the active task. Include read-back and status.
- **Completed tasks** section: brief summaries. Keep last 5-10, archive older.
- **OUTBOX** section: orchestrator drains on "check status" and clears.

### Message archiving
- AGENT-STATUS.md MESSAGES: only latest round. Older messages go to MESSAGE ARCHIVE.

### SHARED.md cleanup
- Mark old entries as `(SUPERSEDED)` instead of deleting.
- Mark bugs as `FIXED` when resolved.

---

## Starting a New Project

1. Copy this folder into your project root
2. Update cold-start messages with project name
3. Create an `agents/[name].md` file for every agent (copy from `_template.md`)
4. Define each agent's DOMAIN OWNERSHIP
5. Reset `AGENT-STATUS.md`
6. Fill in `ROADMAP.md`
7. Start ideation if needed, or assign tasks directly

---

## Scaling the Team

**To add an agent:**
1. Copy `agents/_template.md` → `agents/[name].md`
2. Define DOMAIN OWNERSHIP
3. Add to Agent Roster and AGENT-STATUS.md

**To remove an agent:**
1. Reassign their domain to another agent
2. Remove from roster and status
3. Archive their file

---

## Version History

| Version | Date | Change |
|---------|------|--------|
| 4.0 | 2026-03-29 | Added: closed-loop communication (read-back rule), pre-flight checklist, task lifecycle states, severity levels with autonomy mapping, circuit breaker for failing agents, consensus mode for high-stakes tasks, go/no-go polling. Inspired by distributed systems, biological swarms, NASA mission control, kitchen brigades, ER teams, and open source governance. |
| 3.0 | 2026-03-28 | Tool-agnostic agents, recovery protocol, handoff protocol, decisions protocol, ideation lock. |
| 2.1 | 2026-03-10 | Ideation phase with 6 templates. Gate rule. |
| 2.0 | 2026-03-10 | Refined from ConciergeAI. Generic template. |
| 1.0 | — | Initial protocol |

---

*This protocol was designed for solo founders orchestrating AI agent teams.*
*Inspired by how the best systems in the world coordinate independent agents: consensus algorithms, ant colonies, flight controllers, surgical teams, and open source communities.*
