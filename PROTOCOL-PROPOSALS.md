# PROTOCOL IMPROVEMENT PROPOSALS — 4 Agent Consensus
Date: 2026-03-29

================================================================
AGENT: RESEARCH
================================================================
## OUTBOX

### PROTOCOL IMPROVEMENT PROPOSALS — Distributed Systems Lens

I've analyzed the full Agent Protocol v3.0 against every pattern in COORDINATION-RESEARCH.md. Below are 10 protocol improvements — each with the inspiring pattern, rationale, and the **exact text** to add to AGENT-PROTOCOL.md.

---

## PROPOSAL 1: Circuit Breaker Protocol (New Section)

**Pattern:** Circuit breaker (distributed systems) + Kitchen "in the weeds" backpressure + ER surge protocols

**Rationale:** The current protocol has no mechanism for handling an agent that keeps producing broken output. The Recovery Protocol assumes one failure — it doesn't address repeated failures or cascade prevention. A circuit breaker stops the bleeding.

**Add after "Recovery Protocol" section:**

```markdown
---

## Circuit Breaker Protocol

When an agent produces repeated failures, stop sending tasks. Don't keep retrying the same broken thing.

### Three States

| State | Meaning | Rule |
|-------|---------|------|
| **CLOSED** (normal) | Agent is working. Track failures. | If agent fails 2 tasks in a row → trip to OPEN |
| **OPEN** (halted) | Agent is broken. No new tasks. | Wait one cycle. Then move to HALF-OPEN |
| **HALF-OPEN** (testing) | Send ONE low-risk task as a canary. | If it passes → CLOSED. If it fails → back to OPEN |

### Rules

1. **Two consecutive failures trips the breaker.** A "failure" means: broken code that doesn't run, wrong files edited, task requirements not met, or agent ignoring instructions.
2. **When OPEN:** Cowork notes the agent's status as `CIRCUIT OPEN — [reason]` in AGENT-STATUS.md. No new tasks are assigned. Founder investigates (bad prompt? wrong context? tool issue?).
3. **Canary task:** A small, well-defined task with a known-correct outcome. If the agent handles it correctly, resume normal operations. If not, the breaker stays open.
4. **Never force through a broken agent.** If an agent's circuit breaker trips twice in one session, reassign its domain to another agent or pause that workstream entirely.

### Backpressure Rule

If an agent reports being blocked or overwhelmed, the response is to **reduce input, not demand faster output.** Cowork holds new tasks for that agent until the current blocker is resolved. This applies to the founder too — don't stack tasks on a struggling agent.
```

---

## PROPOSAL 2: Closed-Loop Communication (Replace Communication Protocol)

**Pattern:** Aviation readback/hearback + ER closed-loop communication + Kitchen call-and-response + Film roll protocol

**Rationale:** The current Communication Protocol describes message flow but doesn't require agents to confirm they understood a task before executing. This is the single highest-impact safety improvement across every domain we researched — from cockpits to kitchens, the rule is universal: silence is never acceptable.

**Replace the existing communication flow diagram in "Communication Protocol" with:**

```markdown
## Communication Protocol

Agents cannot talk to each other directly. All communication goes through the founder.

### Closed-Loop Rule

Every task assignment follows three steps:

1. **Assign:** Founder gives the agent a task (via cold-start message + agent file)
2. **Echo back:** Agent reads its task and restates what it will do in its own words — written to the top of its OUTBOX as `ECHO: [restated task]`. This is NOT just "acknowledged" — it must include the specific deliverables, files it will touch, and what "done" looks like.
3. **Confirm or correct:** Founder reads the echo. If correct, says "confirmed" and the agent proceeds. If wrong, founder corrects and the agent echoes again.

**Why:** An agent that misunderstands a task will confidently produce the wrong output. A 30-second echo catches misunderstandings at the cheapest possible moment — before any code is written.

**When to skip:** Simple, unambiguous tasks (e.g., "fix the typo on line 42") don't need a full echo. Use judgment. The rule exists for tasks where misinterpretation is possible.

### Message Flow

```
Founder assigns task (cold-start message)
        |
Agent reads agents/[name].md
        |
Agent writes ECHO to OUTBOX (restated understanding)
        |
Founder confirms or corrects
        |
Agent executes task
        |
Agent updates agents/[name].md — task moved to COMPLETED
        |
Agent writes completion message to OUTBOX
        |
Founder says "check status" to Cowork
        |
Cowork drains OUTBOXes → publishes to AGENT-STATUS.md
```

### The "check status" command
When the founder says **"check status"** to Cowork:
1. Read all `agents/*.md` files
2. Archive old messages in `AGENT-STATUS.md`
3. Drain each `## OUTBOX` and publish new messages
4. Clear each agent's OUTBOX
5. Write next tasks into agent files if needed
6. Return brief status: what was done, what's blocked, what's next
```

---

## PROPOSAL 3: Commander's Intent (Add to Core Rules)

**Pattern:** Military Auftragstaktik + Commander's Intent + NASA flight rules

**Rationale:** The current protocol tells agents WHAT to do but doesn't consistently tell them WHY or what success looks like. When an agent hits an unexpected obstacle, it either gets stuck (reports a blocker) or improvises blindly. Commander's Intent gives agents a fallback — if specific instructions fail, the intent tells them what to aim for.

**Add as Core Rule #12:**

```markdown
12. **Every task carries intent.** When assigning a task, always include: (a) **WHY** — the purpose behind the task, and (b) **DONE LOOKS LIKE** — the specific end state that means success. If an agent can't complete the task as described but can still achieve the intent another way, it should propose the alternative in its OUTBOX — not silently improvise, and not just report "blocked."
```

**Add to the Cold-Start Messages section, update the "Any agent" template:**

```markdown
**Any agent:**
```
You are [AGENT NAME] on [PROJECT NAME]. Read AGENT-PROTOCOL.md, then agents/[name].md for your task, then SHARED.md.

Your task will include WHY (purpose) and DONE LOOKS LIKE (end state). If you can't complete the specific instructions but can still achieve the intent, propose an alternative in your OUTBOX. Execute your task. When done: update your agent file and write to your OUTBOX.
```
```

---

## PROPOSAL 4: Go/No-Go Gate (New Section)

**Pattern:** NASA Go/No-Go polling + Aviation pre-flight checklists + Kitchen "Fire" two-phase commit + Surgical "Time Out"

**Rationale:** The protocol has no formal gate before irreversible operations (deploying to production, running database migrations, pushing to a public repo, deleting data). The Design Gate (Rule #11) only covers visual changes. We need a general-purpose gate for any high-stakes action.

**Add after "Handoff Protocol" section:**

```markdown
---

## Go/No-Go Protocol

Before any irreversible operation, pause and poll.

### What counts as irreversible
- Deploying to production
- Running database migrations
- Pushing to a public repository
- Deleting data or dropping tables
- Sending external communications (emails, notifications)
- Any action that can't be undone with `git checkout`

### The Gate

1. **Agent identifies an irreversible action** in its task and writes to OUTBOX: `GO/NO-GO: [describe what's about to happen and what it affects]`
2. **Cowork** (if active) reviews and adds any concerns
3. **Founder** gives explicit `GO` or `NO-GO`
4. **A single NO-GO stops everything.** No "probably fine." No "let's just try it."

### Pre-Execution Checklist

Before any deploy or migration, the executing agent must verify and report (not just say "yes"):

- [ ] What exactly will change?
- [ ] Is there a rollback path?
- [ ] What could break?
- [ ] Is the current build passing?

Report actual state, not assumed state. "Database backup exists: 2026-03-29T10:00:00Z, 4.2GB" — not just "yes, backup exists."
```

---

## PROPOSAL 5: Heartbeat & Stale State Detection (New Section)

**Pattern:** Raft heartbeats + CRDT vector clocks + Aviation ATIS versioning + Pheromone decay

**Rationale:** The protocol has no way to detect stale state. An agent could be reading an outdated SHARED.md, working from a superseded contract, or operating on assumptions that changed two sessions ago. Every distributed system and biological system has signal decay — information expires without active reinforcement.

**Add to "Housekeeping Rules" section:**

```markdown
### Freshness checks

State goes stale. Every agent must check before acting.

1. **Version your contracts.** Every entry in SHARED.md gets a version tag: `v1`, `v2`, etc. When an agent reads a contract, it notes the version in its task log. If the contract updates, downstream agents know they may need to re-read.
2. **Timestamp your completions.** When marking a task as COMPLETED, include the date: `[2026-03-29] Built the auth API — contract in SHARED.md v2`. This lets anyone see how fresh the work is.
3. **Stale = suspect.** If an agent file hasn't been updated in 3+ sessions and still has an active task, Cowork flags it as potentially stale during "check status." The agent may need a fresh start.
4. **Re-read before depending.** Before using another agent's output, re-read SHARED.md. Don't assume the contract you read two sessions ago is still current. If the version changed, verify your work still aligns.
```

---

## PROPOSAL 6: Span of Control & Scaling Tiers (Replace "Scaling the Team")

**Pattern:** ICS 3-7 span of control + Orchestra conductor threshold (~25) + Kitchen brigade hierarchy + Bulkhead partitioning

**Rationale:** The current "Scaling the Team" section describes mechanics (how to add/remove agents) but not limits. Every high-performance coordination system enforces hard span-of-control limits. The protocol should too.

**Replace "Scaling the Team" section with:**

```markdown
---

## Scaling the Team

### Span of Control Rule

**No orchestrator manages more than 7 agents.** Optimal is 3-5.

| Team Size | Structure |
|-----------|-----------|
| 1-5 agents | Flat: Founder + Cowork manage all agents directly |
| 6-7 agents | Flat but at limit: Consider splitting domains |
| 8-12 agents | Two-tier: Add a second Cowork or team lead agent. Each lead manages 3-6 agents. |
| 13+ agents | Multi-tier: Leads manage sub-teams. One senior Cowork coordinates leads. |

**Why:** An orchestrator tracking 10+ agents loses state. It starts dropping context, giving contradictory instructions, and missing blockers. This isn't a suggestion — it's a cognitive limit that applies to AI agents as much as humans.

### Adding an Agent
1. Copy `agents/_template.md` → rename to `agents/[name].md`
2. Define the agent's domain in the DOMAIN OWNERSHIP section
3. Add a row to the Agent Roster table
4. Add a line to `AGENT-STATUS.md`
5. **Check span of control.** If this pushes any orchestrator past 7 direct reports, restructure before proceeding.

### Removing an Agent
1. Reassign their domain to another agent
2. Remove from Agent Roster and AGENT-STATUS.md
3. Archive or delete their agent file

### The Tournant (Swing Agent)

In a team of specialists, keep one generalist. The **Tournant** is an agent with no fixed domain — it fills in wherever needed: covering a blocked agent's domain, handling overflow, or doing one-off tasks that don't fit any specialist.

- The Tournant reads ALL domain contracts in SHARED.md
- It can write to any domain, but only when assigned by Cowork
- When not needed, it's idle — don't make up work for it
```

---

## PROPOSAL 7: Severity → Autonomy Mapping (Add to Core Rules)

**Pattern:** SRE severity levels + ER triage (ESI) + Military Rules of Engagement + Nuclear positive control

**Rationale:** The current protocol treats all actions equally — agents always need founder review. But fixing a typo and deploying to production are not the same risk level. The protocol should define what agents can do autonomously vs. what requires escalation. Default is safe (positive control — silence means stop, not proceed).

**Add as new section after "Core Rules":**

```markdown
---

## Autonomy Levels

Not every action needs founder approval. But dangerous actions always do.

### The Levels

| Level | Risk | Agent Can... | Examples |
|-------|------|-------------|----------|
| **GREEN** | Reversible, low-risk | Act autonomously, report in OUTBOX | Fix typos, add comments, refactor within own domain, run tests |
| **YELLOW** | Moderate risk, reversible | Act but notify founder in OUTBOX before moving on | Create new files, change shared interfaces, modify build config |
| **RED** | Irreversible or high-impact | Stop and get explicit founder approval (Go/No-Go) | Deploy, migrate DB, delete data, push to public repo, send external comms |

### The Default Rule

**When in doubt, treat it as the higher level.** A YELLOW that might be RED is RED.

### Positive Control

The default agent state is **idle**, not executing. An agent only works when explicitly given a task. If an agent loses context mid-task (session crash, unclear state), it stops and waits — it does not continue guessing. Silence from the founder means "wait," never "proceed."
```

---

## PROPOSAL 8: Diverse Validation for Critical Decisions (New Section)

**Pattern:** Byzantine fault tolerance (3f+1 rule) + Triple Modular Redundancy + Space Shuttle quad redundancy + diverse backup + IV&V

**Rationale:** The protocol currently has no mechanism for validating critical decisions. When a single agent makes an architectural choice or writes security-critical code, there's no second opinion. For decisions that are expensive to reverse, running a second agent with a different perspective catches systematic errors that self-review cannot.

**Add after "Go/No-Go Protocol" section:**

```markdown
---

## Validation Protocol

For critical decisions, one agent's opinion is not enough.

### When to validate

- Architectural choices that affect multiple agents' domains
- Security-critical code (auth, payments, data access)
- Performance-critical paths
- Any decision the founder flags as "validate this"

### How to validate

1. **The producing agent** completes its work and writes the output + rationale to SHARED.md or its OUTBOX.
2. **A different agent** reviews the work independently. The validator must:
   - Be a different agent (not the same agent re-reviewing its own work)
   - Ideally be a different tool/model (Claude reviewing Codex's work, or vice versa) — this catches systematic biases that same-model review misses
   - Define its own acceptance criteria, not just check against the producer's criteria
3. **The validator** writes its assessment to its OUTBOX: `VALIDATED: [what] — [pass/fail] — [findings]`
4. **If the validator finds issues:** the producing agent addresses them. The founder decides if re-validation is needed.

### The Three-Agent Rule (optional, for highest-stakes decisions)

For decisions that are truly irreversible and high-impact, run three independent agents and require 2/3 agreement. Three copies of the same agent with the same prompt is NOT diversity — use different models, different prompts, or different approaches.

**When all three disagree, escalate to the founder.** Don't pick the majority of a 3-way split.
```

---

## PROPOSAL 9: Sterile Mode for Critical Operations (Add to Core Rules)

**Pattern:** Aviation Sterile Cockpit Rule (14 CFR 121.542) + Nuclear positive control + Surgical Time Out

**Rationale:** During critical operations (deploying, migrating data, debugging a production incident), agents should not be multitasking, running side experiments, or "improving" unrelated code. Every high-risk domain enforces this: pilots below 10,000 feet, surgeons during the procedure, nuclear operators during a launch sequence.

**Add as Core Rule #13:**

```markdown
13. **Sterile mode during critical operations.** During deploys, data migrations, production debugging, or any RED-level operation: the executing agent does NOTHING except the critical task. No refactoring side-quests, no "while I'm here" improvements, no unrelated file changes. Cowork does not assign new tasks to other agents that touch the same domain. Focus is total until the critical operation is confirmed complete.
```

---

## PROPOSAL 10: Structured Message Types (Replace free-form OUTBOX messages)

**Pattern:** ICAO standard phraseology + CRDT message types + Aviation ATIS + Cytokine signaling categories + Kitchen call types

**Rationale:** Current OUTBOX messages are free-form natural language. This makes them hard for Cowork to parse consistently and easy to misinterpret. Every high-reliability system uses typed, structured messages. The protocol should define a fixed vocabulary for agent status communication.

**Add to "Housekeeping Rules" section:**

```markdown
### Structured OUTBOX messages

Use these prefixes in OUTBOX messages so Cowork can parse them reliably:

| Prefix | Meaning | Example |
|--------|---------|---------|
| `DONE:` | Task completed successfully | `DONE: Auth API built. Contract in SHARED.md v3.` |
| `BLOCKED:` | Cannot proceed, needs resolution | `BLOCKED: Need DB schema from Backend before I can build queries.` |
| `ECHO:` | Restating understood task (closed-loop) | `ECHO: I will build the login page using the auth contract in SHARED.md v2.` |
| `GO/NO-GO:` | Requesting approval for irreversible action | `GO/NO-GO: About to run migration adding users table to production DB.` |
| `VALIDATED:` | Review of another agent's work | `VALIDATED: Auth middleware — PASS — no issues found.` |
| `FLAG:` | Something seems wrong but not blocking | `FLAG: The API response times are slow (~2s). Not blocking but worth investigating.` |
| `DECISION:` | Requesting a decision be logged | `DECISION: Chose JWT over sessions for auth. Reason: simpler for MVP, no server state.` |

**Rules:**
- Always lead with the prefix. Free-form explanation follows.
- One message per concern. Don't bundle a DONE and a BLOCKED in the same message.
- Include specifics — file names, version numbers, error messages. Never just "it's broken."
```

---

### SUMMARY: Priority Order for Implementation

If incorporating all 10 is too much at once, here's my recommended priority:

1. **Closed-Loop Communication** (Proposal 2) — highest-impact safety improvement, universal across every domain researched
2. **Commander's Intent** (Proposal 3) — makes every task assignment better with minimal protocol weight
3. **Structured Message Types** (Proposal 10) — makes Cowork's job parseable and reliable
4. **Autonomy Levels** (Proposal 7) — stops agents from being either too cautious or too reckless
5. **Circuit Breaker** (Proposal 1) — prevents cascade failures
6. **Go/No-Go Gate** (Proposal 4) — protects against irreversible mistakes
7. **Sterile Mode** (Proposal 13) — prevents distraction during critical ops
8. **Span of Control** (Proposal 6) — essential for scaling past 5-7 agents
9. **Diverse Validation** (Proposal 8) — catches systematic errors
10. **Freshness Checks** (Proposal 5) — prevents stale state drift

================================================================
AGENT: PRODUCT
================================================================
## OUTBOX

### PROPOSAL SET: Biological Systems — 7 Protocol Improvements

Below are 7 concrete protocol additions inspired by biological coordination systems (ant colonies, bee swarms, immune system, neural networks). Each includes the biological pattern, the rationale, and the **exact protocol text** ready to insert into AGENT-PROTOCOL.md.

---

## PROPOSAL 1: Signal Decay Protocol (Pheromone Evaporation)

**Biological pattern:** In every biological coordination system, signals have built-in expiry. Ant pheromones evaporate. Bee dance vigor fades. Cytokines degrade. Synaptic potentials decay. Nothing persists without active reinforcement. This prevents the system from acting on stale information — a trail to depleted food vanishes naturally.

**Problem it solves:** The current protocol has no expiry on OUTBOX messages, task assignments, or shared state. A task assigned 3 sessions ago that was never picked up still looks active. Stale entries in SHARED.md can mislead agents into building against outdated contracts.

**Protocol text to add as new section after "Housekeeping Rules":**

```markdown
## Signal Decay Protocol

All protocol signals have a time-to-live (TTL). Stale signals mislead agents. Nothing persists without reinforcement.

### TTL Rules

| Signal Type | TTL | What Happens on Expiry |
|-------------|-----|----------------------|
| OUTBOX message | 2 Cowork cycles | If Cowork hasn't drained it in 2 "check status" rounds, flag it as `(STALE)` and surface to founder |
| Task assignment | 3 sessions | If an agent hasn't started the task within 3 sessions, Cowork marks it `(EXPIRED)` and returns it to the backlog |
| API contract in SHARED.md | Until superseded | Active contracts stay. When superseded, mark `(SUPERSEDED — [date])`. Contracts not referenced by any active task for 5+ sessions get flagged for founder review |
| Bug report in SHARED.md | 5 sessions | Unfixed bugs older than 5 sessions get escalated to founder with `(AGING)` tag |
| Decision in DECISIONS LOG | Permanent | Decisions don't expire, but Cowork reviews the log every 10 sessions and flags any that may no longer apply |

### The Reinforcement Rule

If a task or signal is still relevant but approaching its TTL, the responsible agent or founder must **actively reaffirm** it — update the timestamp or add `(REAFFIRMED — [date])`. Passive persistence is not enough. If nobody cares enough to reaffirm, the signal was not important.
```

---

## PROPOSAL 2: Quorum Sensing for Irreversible Actions (Bee Swarm + Immune System)

**Biological pattern:** Bees don't commit to a nest site when one scout likes it. They wait until ~15 scouts independently arrive at the same site and confirm its quality. T cells don't launch a full immune response from a single signal — they require antigen presentation + costimulatory confirmation + cytokine context (three independent signals). In both systems, commitment is threshold-based: multiple independent confirmations must converge before irreversible action.

**Problem it solves:** The current protocol has no gate for dangerous operations beyond "when in doubt, stop and ask." There's no structured process for validating irreversible actions like database migrations, deployment, or deleting production data. A single agent can propose and a single founder approval can greenlight something catastrophic.

**Protocol text to add as new section after "Recovery Protocol":**

```markdown
## Quorum Protocol (Irreversible Actions)

Before any irreversible operation, the system requires independent confirmation from multiple sources. A single signal is never enough for destructive action.

### What Counts as Irreversible
- Database migrations or schema changes
- Deployment to production
- Deleting user data or production resources
- Changing authentication or security configuration
- Any action that cannot be undone with `git checkout`

### The Three-Signal Rule

Irreversible actions require THREE independent confirmations before execution:

1. **The proposing agent** states what it wants to do and why, written to its OUTBOX
2. **A second agent or validator** independently verifies the action is safe (e.g., "I reviewed the migration — it's backward-compatible" or "Tests pass against the new schema")
3. **The founder** gives explicit approval after reading both signals

No shortcutting. "The founder told me to do it" is only one signal — the agent must still verify safety independently (signal 1) and another agent or automated check must confirm (signal 2).

### Quorum Threshold Scaling

| Risk Level | Required Signals | Examples |
|-----------|-----------------|----------|
| Reversible | 1 (agent acts) | Code changes on a branch, writing tests, updating docs |
| Difficult to reverse | 2 (agent + founder) | Pushing to remote, changing dependencies, modifying CI |
| Irreversible | 3 (agent + validator + founder) | Production deploy, data migration, security changes |

### The Independence Rule

Confirmations must be **independent** — the same agent running twice is one signal, not two. Different agents examining the same evidence from different angles count as independent. The goal is to catch errors that a single perspective would miss, just as bee scouts each perform their own firsthand evaluation of a nest site.
```

---

## PROPOSAL 3: Two-Tier Response System (Immune System Innate + Adaptive)

**Biological pattern:** The immune system operates on two tiers. The innate system responds in minutes with generic pattern-matching — it recognizes broad threat categories and acts immediately. The adaptive system takes days but produces precise, targeted responses. Critically, the innate system hands off structured context to the adaptive system (dendritic cells carry processed antigen to T cells), and after resolution, the adaptive system improves the innate system's response for next time (antibodies enhance phagocytosis).

**Problem it solves:** The current protocol treats all problems the same way — an agent encounters an issue, reports it, waits for Cowork, waits for founder. A known bug pattern (e.g., "CORS error on new endpoint") goes through the same heavyweight process as a novel architectural problem. There's no fast path for known issues and no mechanism for learning from past resolutions.

**Protocol text to add as new section after "Recovery Protocol":**

```markdown
## Two-Tier Response Protocol

Problems are handled at two speeds. Known patterns get fast responses. Novel situations get careful deliberation. The system learns from each resolution.

### Tier 1: Fast Response (Known Patterns)

Agents maintain a **KNOWN ISSUES** section in SHARED.md — a lookup table of previously encountered problems and their fixes:

```
## KNOWN ISSUES (Tier 1 Responses)

| Pattern | Symptom | Fix | Added By | Date |
|---------|---------|-----|----------|------|
| CORS | "blocked by CORS policy" on new endpoint | Add endpoint to CORS allowlist in config | claude-code | 2026-03-29 |
| DB Connection | "ECONNREFUSED" on local dev | Run `docker compose up -d db` | codex | 2026-03-29 |
```

When an agent encounters a problem matching a Tier 1 pattern:
1. Apply the documented fix immediately — no escalation needed
2. Note in OUTBOX: "Hit known issue [PATTERN NAME], applied Tier 1 fix"
3. If the fix doesn't work, escalate to Tier 2

### Tier 2: Deliberate Response (Novel Problems)

For problems with no matching Tier 1 pattern:
1. Agent documents the problem with full context (what happened, what was expected, what was tried)
2. Agent writes to OUTBOX: "Novel issue — needs Tier 2 analysis"
3. Cowork routes to the appropriate domain agent or flags for founder
4. **After resolution:** The solving agent writes a new Tier 1 entry to KNOWN ISSUES so the next encounter is fast

### The Learning Loop

Every Tier 2 resolution MUST produce a Tier 1 entry. This is how the system gets faster over time — the "adaptive" response trains the "innate" response. If the same novel problem appears twice without a Tier 1 entry being created, that's a process failure.
```

---

## PROPOSAL 4: Stigmergic Task Coordination (Ant Colony)

**Biological pattern:** Ants don't receive task assignments from a manager. They encounter environmental signals (pheromone trails, unattended brood, accumulated waste) and respond based on their own activation thresholds. A forager encountering a strong pheromone trail follows it. A nurse encountering hungry larvae feeds them. The environment IS the task board — tasks advertise themselves through accumulated urgency signals, and agents self-select based on capability. Communication overhead scales with environmental complexity, not agent count.

**Problem it solves:** The current protocol is entirely top-down: founder assigns tasks through Cowork, Cowork writes tasks into agent files, agents execute. This creates a bottleneck at Cowork and the founder — nothing happens without explicit assignment. If the founder is away, agents sit idle even when there's obvious work to do.

**Protocol text to add as enhancement to "Communication Protocol" section:**

```markdown
### Stigmergic Task Board (Self-Selection for Low-Risk Work)

For pre-approved, low-risk task categories, agents may self-select work from a shared task board rather than waiting for explicit assignment. This reduces founder bottleneck for routine work.

#### How It Works

SHARED.md includes a **TASK POOL** section — a list of pre-approved tasks that any qualified agent may pick up:

```
## TASK POOL (Self-Select)

| Task | Domain | Urgency | Claimed By | Status |
|------|--------|---------|------------|--------|
| Fix typo in login error message | frontend | Low | — | Open |
| Add input validation to /api/users | backend | Medium | — | Open |
| Write tests for payment module | backend | Medium | codex | In Progress |
```

#### Self-Selection Rules

1. **Only pre-approved tasks.** The founder or Cowork adds tasks to the TASK POOL. Agents never create tasks for themselves.
2. **Domain match required.** An agent may only claim tasks in its own domain.
3. **Claim before starting.** Write your agent name in "Claimed By" before beginning work. If another agent already claimed it, find a different task.
4. **One claim at a time.** Finish or release your current claim before taking another.
5. **Urgency as pheromone.** Higher-urgency tasks should be picked up first. If a Medium task and a Low task are both available, take the Medium.
6. **Founder override.** A direct task assignment from the founder always supersedes self-selected work. Drop the pool task (mark as "Released") and do the assigned task.

#### What CANNOT Go in the Task Pool
- Anything irreversible (deploy, migrate, delete)
- Architectural decisions
- Work that crosses domain boundaries
- Anything not yet specified in SHARED.md or ROADMAP.md

This is the "ant trail" — low-risk work accumulates visibility in the pool, and idle agents follow the strongest signals without waiting for central dispatch.
```

---

## PROPOSAL 5: Excitatory-Inhibitory Balance — Backpressure Protocol (Neural Networks)

**Biological pattern:** The brain maintains a constant balance between excitatory (glutamate) and inhibitory (GABA) signaling. Every excitatory signal automatically recruits proportional inhibition — this is the "overflow valve" that prevents seizures (runaway cascading activation). Critically, inhibition is *targeted*, not wholesale shutdown. And strategic *disinhibition* — temporarily removing constraints — accelerates progress in specific areas. Without this balance, the system either seizes (too much excitation) or goes silent (too much inhibition).

**Problem it solves:** The current protocol has no mechanism for an overwhelmed agent to signal backpressure. If Cowork assigns a task and the agent is already struggling with the current one, there's no structured way to say "slow down." The protocol also has no concept of throttling — the system pushes work regardless of capacity. This mirrors the immune system's cytokine storm: unchecked positive feedback (more tasks → more urgency → more tasks) destroys the system.

**Protocol text to add as new section after "Core Rules":**

```markdown
## Backpressure Protocol

Agents must be able to signal overload. The response to overload is to REDUCE INPUT, not demand faster output.

### Agent Load Signals

Every agent reports one of three states in their agent file header:

| State | Meaning | What Happens |
|-------|---------|-------------|
| 🟢 GREEN | Ready for work | Normal task assignment |
| 🟡 YELLOW | Working, near capacity | No new tasks until current task completes. Cowork may queue next task but does NOT assign it yet. |
| 🔴 RED | Blocked or failing | All new assignment stops. Cowork investigates. Founder notified. |

### How to Signal

Agents update their state at the top of their agent file:
```
## [AGENT NAME]
- Load: 🟢 GREEN
```

When an agent hits a problem that will delay completion, they immediately update to YELLOW or RED and note the reason. They do NOT silently struggle.

### Cowork's Throttling Rules

1. **Never assign to a RED agent.** Investigate the block first.
2. **Never stack tasks on a YELLOW agent.** Queue the next task in AGENT-STATUS.md but don't write it into the agent file until the agent goes GREEN.
3. **If 50%+ of agents are YELLOW or RED,** flag to founder: "System is overloaded. Recommend reducing scope or extending timeline."
4. **Redistribute, don't escalate pressure.** If Agent A is RED and Agent B is GREEN, consider whether Agent B can take a task from A's queue (respecting domain boundaries).

### The Disinhibition Rule

During time-sensitive work (e.g., a critical bug fix), the founder may temporarily relax non-essential rules to accelerate a specific agent. For example: "Skip the design gate for this hotfix" or "Push directly to the feature branch without review." This must be:
- Explicitly declared by the founder
- Scoped to a specific task (not open-ended)
- Logged in the DECISIONS LOG
- Reversed when the task completes
```

---

## PROPOSAL 6: Immune Memory — Session Knowledge Persistence (Immune System Memory Cells)

**Biological pattern:** After the immune system defeats a pathogen, it doesn't just return to baseline. It creates memory cells at multiple tiers: central memory (lymph nodes — rapid proliferation on re-encounter), effector memory (patrol peripheral tissues — immediate local response), and tissue-resident memory (permanently stationed — fastest response). Each subsequent encounter produces a faster, stronger, more refined response through affinity maturation.

**Problem it solves:** The protocol acknowledges that "AI agents are stateless" and that ".md files are memory." But it doesn't structure WHAT to remember or HOW to make future sessions faster. Every session starts from scratch — reading the same files, re-deriving the same context. There's no mechanism for building up institutional knowledge that makes the 10th session faster than the 1st.

**Protocol text to add as new section after "Housekeeping Rules":**

```markdown
## Session Memory Protocol

Each agent session should be faster than the last. Agents build institutional memory at three tiers.

### Tier 1: Agent-Local Memory (in agents/[name].md)

Each agent file includes a **CONTEXT CACHE** section — a concise summary of hard-won knowledge from past sessions that future sessions need immediately:

```
## CONTEXT CACHE
- Auth uses Supabase RLS, not middleware. Don't add auth middleware.
- The payment module is in /src/lib/payments, NOT /src/api/payments (moved in session 4)
- Build requires `pnpm`, not `npm`. CI will fail on npm.
- The Stripe webhook secret is in .env.local, not .env
```

Rules:
- Max 10 entries. When adding an 11th, remove the least useful one.
- Only record things that COST TIME to rediscover — surprising locations, non-obvious configurations, past mistakes.
- Do NOT duplicate what's in SHARED.md or README.

### Tier 2: Shared Team Memory (in SHARED.md — KNOWN ISSUES table)

Cross-agent knowledge about recurring problems and fixes. (See Two-Tier Response Protocol.)

### Tier 3: Project-Level Memory (in AGENT-STATUS.md — DECISIONS LOG)

Strategic decisions that constrain all future work. Already defined in the protocol — no change needed.

### The Affinity Rule

When an agent encounters a problem it has a CONTEXT CACHE entry for, it should note in its OUTBOX: "Used cached context: [entry]." This signals to Cowork that the cache is working. Entries that are never referenced in 10+ sessions should be pruned — they've either become obvious or irrelevant.
```

---

## PROPOSAL 7: Cross-Inhibition for Decision Deadlocks (Bee Swarm Stop Signals)

**Biological pattern:** When bee scouts discover competing nest sites, they don't just advocate for their preferred site — they actively suppress competing proposals. A scout head-butts dancers advertising rival sites with a 380ms "stop signal," causing them to shorten or stop dancing. Combined with natural dance decay and positive feedback for better sites, this three-mechanism system (amplification + cross-inhibition + decay) resolves deadlocks without any central authority. It's structurally identical to competing neural integrator circuits in the brain.

**Problem it solves:** The current protocol has no process for resolving disagreements between agents or competing approaches. If Agent A proposes approach X and Agent B proposes approach Y, the only resolution is founder arbitration. For technical decisions that don't require strategic judgment, this wastes the founder's time and creates a bottleneck.

**Protocol text to add as new section after "Decisions Protocol":**

```markdown
## Competing Proposals Protocol

When agents propose competing approaches to the same problem, use structured evaluation rather than defaulting to founder arbitration for every disagreement.

### When This Applies
- Two agents propose different technical approaches to the same task
- An agent proposes something that contradicts a previous agent's work
- Multiple valid architectures exist and the team needs to pick one

### The Evaluation Process

1. **Both proposals written.** Each proposing agent writes their approach to SHARED.md under a **PROPOSALS** section with:
   - What the approach is (2-3 sentences max)
   - One key advantage
   - One key risk
   - Estimated effort

2. **Cross-examination.** Each proposing agent reads the competing proposal and writes ONE specific, technical objection (the "stop signal"). Not "I disagree" — a concrete risk or flaw. Written as a reply under the competing proposal.

3. **Decay test.** If either proposer cannot defend against the objection within one session, their proposal is withdrawn. Good proposals survive cross-examination; weak ones fade — just like bee dances that can't maintain vigor.

4. **Quorum or founder.** If one proposal clearly survives and the other is withdrawn, Cowork adopts the survivor. If both survive cross-examination (both are defensible), Cowork summarizes both with the objections and escalates to the founder for a strategic call.

### What This Does NOT Apply To
- Strategic decisions (pricing, positioning, scope) — always founder
- Decisions with irreversible consequences — always founder
- Anything outside the agents' domain expertise — always founder

This is for technical trade-offs where agents have the expertise to evaluate each other's work. The goal is to resolve the 80% of disagreements that are technical, not strategic, without burning founder attention.
```

---

### SUMMARY: All 7 Proposals

| # | Proposal | Biological Pattern | Section to Add |
|---|----------|--------------------|----------------|
| 1 | Signal Decay Protocol | Pheromone evaporation (ants), dance decay (bees), cytokine degradation (immune), synaptic decay (neural) | New section after Housekeeping Rules |
| 2 | Quorum Protocol | Bee swarm quorum sensing, immune two-signal requirement, neural threshold commitment | New section after Recovery Protocol |
| 3 | Two-Tier Response | Immune innate + adaptive system | New section after Recovery Protocol |
| 4 | Stigmergic Task Board | Ant colony pheromone-based task selection | Enhancement to Communication Protocol |
| 5 | Backpressure Protocol | Neural excitatory-inhibitory balance, immune Treg suppression | New section after Core Rules |
| 6 | Session Memory Protocol | Immune memory cells (central, effector, tissue-resident) | New section after Housekeeping Rules |
| 7 | Competing Proposals Protocol | Bee swarm cross-inhibition + dance decay | New section after Decisions Protocol |

**Recommendation to founder:** Proposals 1 (Signal Decay), 5 (Backpressure), and 6 (Session Memory) are the highest-impact, lowest-risk additions — they solve real pain points without changing the protocol's core architecture. Proposals 2 (Quorum) and 3 (Two-Tier Response) add safety and speed respectively. Proposals 4 (Stigmergic Task Board) and 7 (Competing Proposals) are more ambitious — they reduce founder bottleneck but require trust in agent self-governance.

================================================================
AGENT: BACKEND
================================================================
## OUTBOX

### PROPOSAL 1: DOMAIN OWNERSHIP FILE (OWNERS Protocol)
**Inspired by:** Linux MAINTAINERS file, Kubernetes OWNERS files, GitHub CODEOWNERS

**Problem it solves:** The current protocol says "define each agent's domain in their `.md` file under DOMAIN OWNERSHIP" but there's no machine-readable, centralized ownership map. When a file changes, there's no way to know who should review it without reading every agent file.

**Add new section to AGENT-PROTOCOL.md after "Role Boundaries":**

```markdown
### OWNERS File

Maintain an `OWNERS.md` file in the project root that maps every file/directory to its owning agent. This is the single source of truth for domain boundaries.

**Format:**
| Path Pattern | Owner | Reviewer | Notes |
|---|---|---|---|
| `app/api/**` | claude-code | codex | Backend API routes |
| `app/components/**` | gemini | claude-code | Frontend components |
| `lib/db/**` | claude-code | — | Database layer |
| `SHARED.md` | cowork | — | Cowork owns; all agents read |

**Rules:**
1. Every file in the project must match at least one pattern. Unmatched files are owned by the founder.
2. Last matching pattern wins (like `.gitignore`).
3. Only the **Owner** may write to files matching the pattern. **Reviewer** is consulted before changes merge.
4. If a task requires touching files owned by another agent, the requesting agent writes the spec to SHARED.md and the owning agent makes the change.
5. Cowork updates `OWNERS.md` when domains change. Changes require founder approval.
6. When a new file is created, the creating agent must add it to `OWNERS.md` in their OUTBOX message.
```

---

### PROPOSAL 2: PROVENANCE CHAIN (Signed-off-by Protocol)
**Inspired by:** Linux kernel patch tags (Signed-off-by, Reviewed-by, Acked-by, Tested-by)

**Problem it solves:** Currently there's no audit trail for who created, reviewed, or approved a change. When bugs surface later, there's no traceability.

**Add new section to AGENT-PROTOCOL.md after "Handoff Protocol":**

```markdown
## Provenance Protocol

Every completed task must carry a provenance chain in the agent's OUTBOX message. This creates an audit trail that survives across sessions.

**Tags:**
| Tag | Meaning | Required? |
|---|---|---|
| `Created-by: [agent]` | Agent that produced the work | Always |
| `Reviewed-by: [agent]` | Agent that reviewed for correctness | For code changes |
| `Approved-by: founder` | Founder approved the change | For merges to main |
| `Tested-by: [agent/founder]` | Who verified the work | Always |

**OUTBOX format with provenance:**
```
Task: Build user authentication API
Branch: feat/auth
Files changed: app/api/auth.ts, lib/db/users.ts
Created-by: claude-code
Tested-by: claude-code (unit tests pass, manual test screenshots attached)
```

**Rules:**
1. No agent can be both `Created-by` and `Reviewed-by` on the same task. If only one builder agent exists, founder reviews.
2. `Approved-by: founder` is added when the founder merges the branch. Not before.
3. If a change is substantially reworked after review, the `Reviewed-by` tag is removed — the reviewer must re-review.
4. Cowork copies provenance chains into AGENT-STATUS.md during "check status" for historical record.
```

---

### PROPOSAL 3: INTEGRATION TESTING PROTOCOL (Merge Queue Pattern)
**Inspired by:** Bors merge bot, GitHub merge queues, linux-next integration tree, the "Not Rocket Science Rule"

**Problem it solves:** Two agents can each produce working code independently that breaks when combined. The current protocol has no mechanism to catch this — the founder discovers conflicts at merge time.

**Add new section to AGENT-PROTOCOL.md under "Parallel Work Protocol":**

```markdown
### Integration Testing Rule (The "Not Rocket Science" Rule)

> Main must always pass all tests. Every change is tested against the combined state of all accepted work before merging.

When multiple agents work in parallel on separate branches:

1. **Before merging any branch**, the founder (or a designated agent) must:
   - Create a temporary integration branch from main
   - Merge ALL pending agent branches into it
   - Run the full test suite against the combined result
   - Only if tests pass, merge branches to main one at a time

2. **If integration fails:**
   - Identify which combination of changes conflicts
   - The agent whose domain contains the broken code fixes it
   - Re-run integration test
   - Never merge a branch that hasn't been tested against current main + all other pending branches

3. **Merge order matters.** Merge the branch with the most dependencies first (e.g., database schema before API before frontend). Cowork determines merge order during "check status."

4. **After each merge to main**, all other pending branches must rebase and re-verify. A branch that was green before a merge may be red after.
```

---

### PROPOSAL 4: SEVERITY → AUTONOMY MAPPING
**Inspired by:** PagerDuty/Google SRE severity levels, incident response escalation matrices

**Problem it solves:** Core Rule #8 says "When in doubt, stop and ask" but doesn't define WHEN agents should doubt. The line between "handle it" and "escalate" is undefined, leading to either constant escalation (slow) or silent improvisation (dangerous).

**Replace or augment Core Rule #8 with:**

```markdown
### Severity-Based Autonomy

Agents classify situations by severity and respond accordingly:

| Severity | Definition | Agent Response |
|---|---|---|
| **SEV-4: Routine** | Expected behavior, minor decisions | Agent handles autonomously. Log in OUTBOX. |
| **SEV-3: Notable** | Unexpected but non-breaking. Missing docs, minor dependency issue, ambiguous spec detail. | Agent handles but explicitly notes it in OUTBOX with rationale. |
| **SEV-2: Significant** | Could break other agents' work, affects architecture, scope ambiguity, anything touching production data. | Agent STOPS. Writes situation + options + recommendation to OUTBOX. Waits for founder decision. |
| **SEV-1: Critical** | Build is broken, data loss risk, security issue, blocked with no workaround. | Agent STOPS immediately. Writes to OUTBOX with `[SEV-1]` prefix. Does not attempt fixes without founder approval. |

**The escalation rule: When in doubt, treat it as one level higher.** A maybe-SEV-3 is a SEV-2. A maybe-SEV-2 is a SEV-1.

**What always requires escalation (SEV-1 or SEV-2):**
- Changing the database schema in ways that aren't additive
- Modifying another agent's domain files
- Adding new dependencies to the project
- Any operation that can't be undone with `git checkout`
- Deviating from the spec in SHARED.md
```

---

### PROPOSAL 5: TASK LIFECYCLE WITH EXPLICIT STATES
**Inspired by:** CI/CD pipeline states, Kubernetes pod lifecycle, GitHub Actions job states

**Problem it solves:** Current task tracking is freeform text. An agent's status is either "doing something" or "done." There's no structured way to know if an agent is blocked, waiting for review, or in what phase of work.

**Add to AGENT-PROTOCOL.md, replacing the freeform "Current task" guidance in Housekeeping:**

```markdown
### Task Lifecycle

Every task in an agent's file must have an explicit status from this state machine:

```
ASSIGNED → ACKNOWLEDGED → IN_PROGRESS → VERIFY → DONE
                ↓              ↓
             BLOCKED        BLOCKED
```

| Status | Meaning | Who transitions |
|---|---|---|
| `ASSIGNED` | Task written into agent file by Cowork | Cowork |
| `ACKNOWLEDGED` | Agent has read the task and confirmed understanding | Agent (on session start) |
| `IN_PROGRESS` | Agent is actively working | Agent |
| `BLOCKED` | Agent cannot proceed. Must include: what's blocking, what's needed, which agent/resource is involved. | Agent |
| `VERIFY` | Work is complete, agent is self-testing | Agent |
| `DONE` | Work is complete and verified. Results in OUTBOX. | Agent |

**Format in agent file:**
```
## CURRENT TASK
- Task: Build the /api/users endpoint per SHARED.md contract
- Status: IN_PROGRESS
- Branch: feat/users-api
- Blocked-by: (none)
- Started: 2026-03-29
```

**Rules:**
1. An agent must move to `ACKNOWLEDGED` before doing any work. This forces re-reading the task spec.
2. `BLOCKED` requires a specific description — "blocked" alone is not valid. Include what you need and from whom.
3. `VERIFY` means the agent is running its own tests. The agent must include test results in the OUTBOX when moving to `DONE`.
4. Cowork checks for stale statuses: any task in `IN_PROGRESS` for more than one session without an OUTBOX update should be flagged.
```

---

### PROPOSAL 6: FAN-OUT / FAN-IN TASK GRAPHS
**Inspired by:** GitHub Actions `needs` keyword, CI/CD DAGs, build system dependency graphs

**Problem it solves:** The current protocol says "Agents build in dependency order" but doesn't formalize how to express or manage dependencies. Cowork has to track this mentally.

**Add to AGENT-PROTOCOL.md under "Implementation Pipeline":**

```markdown
### Task Dependency Graph

When Cowork assigns tasks for a phase, they must be expressed as a dependency graph in AGENT-STATUS.md:

```
## TASK GRAPH — Phase 1

[DB-SCHEMA] claude-code — Create database tables
  ↓
[API-AUTH] claude-code — Build auth endpoints (needs: DB-SCHEMA)
[API-USERS] codex — Build user endpoints (needs: DB-SCHEMA)
  ↓         ↓
[FRONTEND] gemini — Build UI (needs: API-AUTH, API-USERS)
  ↓
[INTEGRATION] founder — Merge and test all branches
```

**Rules:**
1. Tasks with no dependencies (`needs: none`) can start immediately and run in parallel.
2. A task cannot move to `ASSIGNED` until ALL its dependencies are `DONE`.
3. Each task gets a short slug (e.g., `DB-SCHEMA`, `API-AUTH`) used as a reference everywhere — in OUTBOX messages, SHARED.md contracts, and branch names.
4. Cowork updates the graph during "check status" — marking completed tasks, unblocking downstream tasks, and adjusting if priorities change.
5. If a task's dependency fails or changes scope, all downstream tasks are flagged for re-evaluation.
```

---

### PROPOSAL 7: PROGRESSIVE TRUST (Agent Capability Tiers)
**Inspired by:** Kubernetes membership ladder (Member → Reviewer → Approver → Owner), Apache merit system, Linux kernel maintainer progression

**Problem it solves:** A brand new agent file gets the same permissions as a battle-tested one. There's no concept of earning broader autonomy through demonstrated reliability.

**Add to AGENT-PROTOCOL.md under "Scaling the Team":**

```markdown
### Agent Trust Tiers

New agents start with limited autonomy and earn broader permissions through demonstrated reliability.

| Tier | Name | Permissions | How to reach |
|---|---|---|---|
| **T1** | New | Can only work on explicitly assigned tasks. All output reviewed by founder before merge. Cannot modify shared files. | Default for new agents. |
| **T2** | Trusted | Can make minor decisions (SEV-4) autonomously. Output reviewed by founder, but with lighter touch. Can read SHARED.md contracts. | 3+ tasks completed successfully without rework. |
| **T3** | Established | Can make SEV-3 decisions autonomously. Can write to SHARED.md (own sections only). Can suggest tasks in OUTBOX. | 10+ tasks completed. Zero SEV-1 incidents caused. |
| **T4** | Senior | Can review other agents' OUTBOX for correctness. Can propose architectural changes. Broader domain ownership. | Founder's judgment based on sustained quality. |

**Rules:**
1. Trust tier is noted in the agent's `.md` file header.
2. Tier promotion requires founder approval and is logged in the DECISIONS LOG.
3. A SEV-1 incident caused by an agent drops them one tier. Recovery requires 3 clean tasks.
4. Trust does not transfer across projects — each project starts fresh at T1.
5. This is guidance for the founder, not a rigid system. Skip tiers when the situation warrants it.
```

---

### PROPOSAL 8: PRE-FLIGHT CHECKLIST (Mise en Place for Agents)
**Inspired by:** Aviation challenge-response checklists, kitchen mise en place, film pre-production, CI/CD environment validation

**Problem it solves:** Agents jump straight into coding without verifying their prerequisites. They discover missing context, stale contracts, or broken dependencies mid-task — wasting a session.

**Add to Core Rules or as a new section:**

```markdown
### Pre-Flight Checklist

Before starting any task, every agent must verify these items and log confirmation in their agent file:

1. **Read AGENT-STATUS.md** — Confirm you have the latest context. Note the current phase.
2. **Read your task spec** — If it references a SHARED.md contract, verify the contract exists and is current (not marked SUPERSEDED).
3. **Check dependencies** — If your task has upstream dependencies (per the Task Graph), verify they are `DONE`. If not, you are `BLOCKED`.
4. **Check your branch** — Verify you're on the correct feature branch, not main. If the branch doesn't exist, create it from main.
5. **Verify build passes** — Run the project's test suite before making changes. If it's already broken, report `[SEV-1]` in OUTBOX. Do not start work on a broken build.

**Format in agent file:**
```
## PRE-FLIGHT
- [x] AGENT-STATUS.md read — Phase 1, no blockers
- [x] Task spec confirmed — contract in SHARED.md section 3.2
- [x] Dependencies met — DB-SCHEMA is DONE (claude-code, 2026-03-29)
- [x] Branch: feat/api-users created from main @ abc1234
- [x] Build passing — all 47 tests green
```

If any item fails, the agent moves to `BLOCKED` and writes the failure to OUTBOX. **Do not start work with a failed pre-flight.**
```

---

### PROPOSAL 9: POSTMORTEM PROTOCOL
**Inspired by:** Google SRE blameless postmortems, Five Whys, incident retrospectives

**Problem it solves:** The Recovery Protocol tells agents how to fix broken things, but doesn't capture WHY things broke. The same failure patterns repeat across sessions because lessons aren't persisted.

**Add to AGENT-PROTOCOL.md after "Recovery Protocol":**

```markdown
## Postmortem Protocol

When a task fails, produces a bug that reaches the founder, or causes a SEV-1/SEV-2 incident, a postmortem is required.

**Who writes it:** The agent whose domain contains the failure, in their OUTBOX. Cowork compiles it into AGENT-STATUS.md during "check status."

**Template:**
```
### POSTMORTEM: [TASK-SLUG] — [date]
**What happened:** One sentence.
**Timeline:** Key events in order.
**Root cause:** Keep asking "why" until you reach a systemic cause, not an individual mistake.
**Contributing factors:** What else made this possible? (Missing test? Unclear spec? Stale contract?)
**What we'll change:**
- [ ] Action item 1 (owner: [agent], deadline: [date])
- [ ] Action item 2 (owner: [agent], deadline: [date])
```

**Rules:**
1. Postmortems are **blameless.** Root cause is never "agent X made a mistake." It's "the system allowed the mistake." Focus on: missing validation, unclear specs, untested paths, missing contracts.
2. Every postmortem must have at least one action item that changes the system (not just "be more careful").
3. Action items are tracked in AGENT-STATUS.md until completed.
4. Before starting a new phase, Cowork reviews outstanding postmortem action items. Unresolved items from the previous phase must be addressed or explicitly deferred by the founder.
```

---

### PROPOSAL 10: TIERED REVIEW PROTOCOL
**Inspired by:** Google's "Three Approval Bits" (LGTM + ownership + readability), two-phase Kubernetes review (lgtm + approve), code review research showing 85% of comments are style vs 15% defects

**Problem it solves:** Core Rule #9 says "Verify your own work" but self-review is weak — the same blind spots that created a bug won't catch it. There's no protocol for cross-agent review.

**Add to AGENT-PROTOCOL.md, augmenting Core Rule #9:**

```markdown
### Review Protocol

Agent output is reviewed at two tiers before merging:

**Tier 1: Self-Verification (always required)**
The producing agent runs automated checks against their own work:
- Tests pass
- No lint/type errors
- Spec in SHARED.md is satisfied
- Results documented in OUTBOX with evidence (test output, screenshots, etc.)

**Tier 2: Cross-Review (required for SEV-2+ changes or multi-file changes)**
A second agent or the founder reviews the work:
- The reviewer checks: Does this satisfy the spec? Does it break any contracts in SHARED.md? Does it conflict with other agents' pending work?
- Reviewer writes `Reviewed-by: [agent]` in the producing agent's OUTBOX (or founder notes approval)
- If the reviewer identifies issues, they write them to their own OUTBOX with `REVIEW: [TASK-SLUG] — [issue]`. The producing agent addresses them in the next session.

**What triggers Tier 2:**
- Changes to shared interfaces or contracts
- Database schema changes
- Changes touching 3+ files
- Any change the producing agent is uncertain about (self-reported)
- Founder can always request Tier 2 for any change

**What skips Tier 2:**
- Single-file bug fixes with passing tests
- Documentation-only changes
- Changes fully covered by automated tests with >80% coverage of the change
```

================================================================
AGENT: DESIGNER
================================================================
## OUTBOX

### PROTOCOL IMPROVEMENT PROPOSALS — Human Organizations Lens

I analyzed five high-performance human coordination systems (ER teams, film production, kitchen brigades, newsrooms, orchestras) cross-referenced with the aviation, military/NASA, distributed systems, and biological research from the other agents. Below are 8 specific protocol additions, each with the inspiring pattern, rationale, and exact text to add to AGENT-PROTOCOL.md.

---

## PROPOSAL 1: Closed-Loop Communication Protocol
**Inspired by:** ER closed-loop communication (Send → Read-back → Confirm), kitchen brigade call-and-response ("Oui, Chef!"), film roll protocol ("Roll sound!" → "Sound speed!"), aviation readback/hearback (FAA 7110.65)

**Rationale:** This pattern appears in every single high-performance human system we researched — ER, kitchen, film, aviation, military. It is the single most universal coordination pattern across all domains. The current protocol has agents write to OUTBOX when done, but there is no confirmation that tasks were understood before execution begins. Misunderstood tasks are the most expensive failure mode because the agent does the wrong work entirely.

**Where to add:** New section after "Core Rules", before "Handoff Protocol"

**Exact protocol text:**

```markdown
## Closed-Loop Communication

Every task assignment follows a three-step loop. No exceptions.

1. **Assign.** Founder gives agent a task (written in agent's CURRENT TASK section).
2. **Echo back.** Before starting work, the agent writes a one-line restatement of the task in its own words at the top of its CURRENT TASK section, prefixed with `> Understood:`. This is NOT a copy-paste — it's the agent's interpretation. Include the specific deliverable and any constraints.
3. **Confirm or correct.** Founder reads the echo. If correct, agent proceeds. If wrong, founder corrects and the loop restarts.

**Why this matters:** An agent that says "acknowledged" has proven nothing about comprehension. An agent that restates the task in its own words reveals whether it actually understood. This catches misunderstandings at the cheapest possible moment — before any work begins.

**The rule:** Silence is never confirmation. Every assignment must be echoed back. If an agent's echo is missing, assume the task was not received.
```

---

## PROPOSAL 2: Two-Phase Task Commit ("Ordering" / "Fire")
**Inspired by:** Kitchen brigade two-phase call system — "Ordering" (awareness, stations note but DON'T cook) → "Fire" (action, begin NOW). Also maps to film's "Picture is up!" (awareness) → "Action!" (execution), and orchestra's cueing protocol (eye contact → preparatory gesture → downbeat).

**Rationale:** The current protocol has a single state: agent receives task and starts working. But in every high-performance human system, there's a gap between awareness and execution. This gap lets agents prepare (load context, read dependencies, check SHARED.md) without the pressure of execution. It also lets the founder stage multiple agents' tasks before triggering parallel execution.

**Where to add:** Addition to the "Communication Protocol" section, after the existing flow diagram

**Exact protocol text:**

```markdown
### Two-Phase Task Commit

Tasks have two phases: **Ordering** and **Fire**.

**Phase 1 — Ordering:** Founder writes the task into the agent's CURRENT TASK section. The task is prefixed with `[ORDERING]`. The agent reads the task, loads context, reads SHARED.md and any dependencies, and writes its echo-back (see Closed-Loop Communication). The agent does NOT begin producing output yet.

**Phase 2 — Fire:** Founder changes the prefix to `[FIRE]` (or sends the agent a cold-start message). The agent begins execution.

**Why two phases:**
- Lets the founder stage tasks for multiple agents simultaneously before triggering parallel work
- Gives agents time to identify blockers BEFORE committing to execution
- Prevents premature execution on incomplete dependencies
- Maps to kitchen "ordering/fire", film "picture is up/action", orchestra "cue/downbeat"

**Shortcut:** For simple tasks with no dependencies, the founder can skip Phase 1 and write the task as `[FIRE]` directly. The echo-back still happens — the agent just doesn't wait for a separate trigger.
```

---

## PROPOSAL 3: Hands-Off Orchestrator Rule
**Inspired by:** ER team leader (hands-off — does NOT perform procedures), kitchen expeditor (calls orders and checks quality but doesn't cook), film 1st AD (coordinates but doesn't operate equipment), orchestra conductor (doesn't play an instrument).

**Rationale:** In every high-performance human system, the coordinator role is explicitly prohibited from doing the work. The moment a coordinator starts building, they lose situational awareness and the whole system degrades. The current protocol defines Cowork as "Chief of Staff / PM" but doesn't explicitly prohibit Cowork from building.

**Where to add:** Addition to the "Agent Roster" section, under "Role Boundaries"

**Exact protocol text:**

```markdown
### The Orchestrator Rule

The orchestrator (Cowork) **never builds.** Cowork's job is:
- Maintain system state (AGENT-STATUS.md)
- Sequence tasks and manage dependencies
- Drain outboxes and route messages
- Identify blockers and escalate to founder

Cowork does NOT: write code, create designs, fix bugs, or produce any deliverable that belongs to another agent's domain. The moment the orchestrator starts doing execution work, they lose the bird's-eye view and the whole team loses coordination.

**The expeditor principle:** In a professional kitchen, the expeditor stands at the pass, calls orders, checks quality, and orchestrates timing — but never touches a pan. Cowork is the expeditor.
```

---

## PROPOSAL 4: Mise en Place Phase (Pre-Execution Readiness Check)
**Inspired by:** Kitchen mise en place ("everything in its place" — all prep done before service), film pre-production (script breakdown + stripboard before shooting), orchestra individual preparation (parts distributed 2+ weeks early, musicians arrive prepared), aviation challenge-response checklists.

**Rationale:** The current protocol assumes agents are ready to execute once they receive a task. But agents are stateless — every session starts fresh. They need to load context, verify dependencies, and confirm their environment is ready. Making this explicit prevents the most common failure: an agent starts building against stale or missing contracts.

**Where to add:** New section after "Two-Phase Task Commit" or as an addition to Core Rules

**Exact protocol text:**

```markdown
### Mise en Place (Pre-Execution Readiness)

Before an agent begins execution (before or during the Ordering phase), it runs through this checklist:

1. **Read your file.** `agents/[name].md` — current task, completed work, any notes from previous sessions.
2. **Read shared state.** `AGENT-STATUS.md` — what other agents are doing, any blockers.
3. **Read contracts.** `SHARED.md` — any API contracts, architecture notes, or specs relevant to your task.
4. **Check dependencies.** If your task depends on another agent's output, verify that output exists. If it doesn't, you are BLOCKED — say so in your OUTBOX. Do not improvise.
5. **Confirm environment.** If your task requires running code, verify the build passes before making changes.

**Report readiness.** After completing the checklist, write `> Ready: [dependencies confirmed / blocked on X]` in your CURRENT TASK section.

**The principle:** Front-load all preparation so execution-time complexity is minimized. During execution, agents should be building — not hunting for context, discovering missing dependencies, or guessing at specs.
```

---

## PROPOSAL 5: Backpressure Protocol
**Inspired by:** Kitchen overload protocol (expeditor holds tickets, host stops seating, Tournant jumps in), ER surge protocols (3 tiers: conventional → contingency → crisis), newsroom deadline management (stories held for next edition rather than rushed), distributed systems circuit breaker pattern.

**Rationale:** The current protocol has no mechanism for what happens when an agent is overwhelmed, blocked, or producing errors. In every human system, the response to overload is to REDUCE INPUT, not demand faster output. Without this, a founder will keep assigning tasks to a broken or overwhelmed agent, compounding failures.

**Where to add:** New section after "Recovery Protocol"

**Exact protocol text:**

```markdown
## Backpressure Protocol

When the system is under stress, reduce input — don't demand faster output.

### Agent Reports Overload
If an agent's task is significantly larger than expected, or if cascading issues are making progress impossible:
1. Agent writes to OUTBOX: `BLOCKED: [reason]` or `OVERLOADED: [reason]`
2. Founder does NOT assign additional tasks to that agent
3. Founder either: (a) breaks the task into smaller pieces, (b) reassigns part of the work, or (c) descopes

### Agent Producing Errors
If an agent's output is broken or incorrect on consecutive attempts:
1. **Stop.** Do not send the same task again.
2. **Diagnose.** Is it a context problem? A dependency issue? A task that's too ambiguous?
3. **Reset.** Start a fresh session with a clearer task definition.
4. If the same agent fails 3 times on the same task, reassign to a different agent or tool.

### System-Wide Overload
If multiple agents are blocked or the founder can't keep up with reviewing output:
1. **Throttle.** Reduce to one active agent at a time.
2. **Finish before starting.** Complete review of existing work before assigning new tasks.
3. **Descope.** Cut the lowest-priority items from the current phase.

**The kitchen rule:** When a cook is "in the weeds," the expeditor holds new tickets, the Tournant jumps in, and the host stops seating. The system throttles at every level. Never push harder on an overloaded system.
```

---

## PROPOSAL 6: Task Slugs
**Inspired by:** Newsroom slug system (human-readable 1-4 word task IDs like "HURRICANE IAN" or "MAYOR RACE" that persist across the entire lifecycle of a story), film slate numbers, kitchen ticket numbers.

**Rationale:** The current protocol refers to tasks by description, which changes as tasks move through stages. A persistent, memorable identifier for each task makes it easy to reference work across agent files, OUTBOX messages, AGENT-STATUS.md, and git branches. It also makes the "check status" flow much faster — the founder can say "status on AUTH-FLOW" instead of re-describing the task.

**Where to add:** Addition to "Core Rules" or "Housekeeping Rules"

**Exact protocol text:**

```markdown
### Task Slugs

Every task gets a **slug** — a human-readable 1-4 word identifier in CAPS that persists across its entire lifecycle.

**Format:** `SLUG-NAME` (1-4 words, hyphenated, all caps)
**Examples:** `AUTH-FLOW`, `LANDING-PAGE`, `DB-SCHEMA`, `ONBOARDING-WIZARD`

**Where slugs appear:**
- In the agent's CURRENT TASK section: `[FIRE] AUTH-FLOW: Build the login and signup API endpoints`
- In OUTBOX messages: `AUTH-FLOW: Complete. API contracts in SHARED.md.`
- In AGENT-STATUS.md: `AUTH-FLOW — Claude Code — Complete`
- In git branches: `feat/auth-flow`

**Why slugs:** When you have 5 agents and 12 tasks, descriptions drift. Slugs don't. "What's the status on AUTH-FLOW?" is faster and less ambiguous than re-describing the task. Every human coordination system with high throughput uses short persistent identifiers — newsroom slugs, kitchen ticket numbers, film slate numbers, military operation names.
```

---

## PROPOSAL 7: Quality Gate at Output
**Inspired by:** Kitchen pass (every plate inspected by expeditor before leaving the kitchen), newsroom copy desk (multi-stage editing pipeline + senior editor sign-off), film dailies review ("check the gate" after each take), NASA IV&V (independent verification and validation), aviation cross-checking.

**Rationale:** The current protocol has agents update their OUTBOX when done, and the founder reviews. But there's no explicit quality gate defined — no checklist of what "done" means, no requirement for agents to verify their own work before declaring completion. Rule 9 says "verify your own work" but doesn't specify how.

**Where to add:** Addition to "Core Rules" or new section after "Handoff Protocol"

**Exact protocol text:**

```markdown
## Quality Gate

No agent output is "done" until it passes the gate. Before writing to OUTBOX:

### Self-Check (Every Agent, Every Task)
1. **Does it work?** If you wrote code, did you run it? If you wrote a spec, is it internally consistent?
2. **Does it match the task?** Re-read your CURRENT TASK (and the echo-back). Did you deliver what was asked?
3. **Does it match the contracts?** If your work depends on or produces an API contract in SHARED.md, does your output match the spec?
4. **Did you break anything?** If you changed existing code, did existing functionality survive?

### Proof of Work
In your OUTBOX message, include **evidence** that the gate passed:
- Code tasks: "Tests pass. Server starts. Login flow returns 200."
- Design tasks: "Screenshot in designs/. Matches design system colors/spacing."
- Spec tasks: "Covers all endpoints from ROADMAP Phase 1. No open questions."

**The pass rule:** In a professional kitchen, every plate crosses the pass — the expeditor inspects temperature, presentation, completeness. A cook who says "it's done" without the plate on the pass has not finished. An agent who says "complete" without verification evidence has not finished.
```

---

## PROPOSAL 8: Operating Modes (Normal / Surge / Crisis)
**Inspired by:** ER surge protocols (3 tiers: conventional → contingency → crisis, each with explicit tradeoffs), kitchen brigade overload response ("in the weeds" is a named state with a defined protocol), newsroom breaking news mode (same structure, compressed tempo), military DEFCON levels.

**Rationale:** The current protocol assumes one operating mode. But real projects hit crunch time, demo deadlines, and emergencies. High-performance human teams don't just "work harder" — they deliberately shift to a named mode with explicit tradeoffs. Naming the mode makes the quality/speed tradeoff conscious rather than accidental.

**Where to add:** New section after "Parallel Work Protocol"

**Exact protocol text:**

```markdown
## Operating Modes

The team operates in one of three named modes. The founder sets the mode. Agents adjust their behavior accordingly.

### NORMAL (Default)
- Full quality gates. Full review cycles.
- Agents work at sustainable pace with proper testing.
- Founder reviews all output before merging.

### SURGE (Deadline Pressure)
- Activated by founder when a deadline requires faster throughput.
- Quality gates still apply, but review cycles compress: founder reviews in batches, not one-by-one.
- Agents may work on tasks in parallel even if there's minor overlap — founder resolves conflicts.
- Testing focuses on critical paths only. Edge cases are logged as TODOs, not blocked on.
- **Explicit tradeoff:** Speed over polish. Ship working code, refine later.

### CRISIS (Something Is Broken)
- Activated when production is down, a demo is imminent, or a critical blocker threatens the project.
- **Single focus.** All agents work on the crisis. Non-critical tasks are paused.
- **Shortest path.** Agents take the fastest path to resolution, even if it's not the cleanest.
- **Founder in the loop continuously.** No waiting for "check status" — direct communication.
- **Exit criteria required.** Founder declares when crisis mode ends and normal mode resumes. Without explicit exit, crisis mode tends to become permanent (and quality degrades indefinitely).
- **Explicit tradeoff:** Working now over clean later. But log every shortcut as tech debt in SHARED.md.

**The rule:** Never accidentally operate in crisis mode. If you're cutting corners, name it. If you can't name the mode, you're in NORMAL — follow full quality gates.
```

---

### SUMMARY OF ALL 8 PROPOSALS

| # | Proposal | Inspired By | Impact |
|---|----------|-------------|--------|
| 1 | **Closed-Loop Communication** | ER, Kitchen, Film, Aviation | Catches misunderstandings before work begins — highest-impact single improvement |
| 2 | **Two-Phase Task Commit** | Kitchen ordering/fire, Film roll protocol, Orchestra cue/downbeat | Prevents premature execution; enables parallel staging |
| 3 | **Hands-Off Orchestrator** | ER team leader, Kitchen expeditor, Film 1st AD, Orchestra conductor | Prevents coordinator from losing situational awareness |
| 4 | **Mise en Place** | Kitchen prep, Film pre-production, Orchestra individual preparation | Eliminates "building against stale context" failures |
| 5 | **Backpressure Protocol** | Kitchen overload protocol, ER surge, Newsroom deadline mgmt | Prevents cascading failures from overwhelmed agents |
| 6 | **Task Slugs** | Newsroom slugs, Film slates, Kitchen ticket numbers | Persistent task identity across the entire lifecycle |
| 7 | **Quality Gate** | Kitchen pass, Newsroom copy desk, Film dailies, NASA IV&V | No output ships without verification evidence |
| 8 | **Operating Modes** | ER surge tiers, Kitchen "in the weeds", Newsroom breaking news, DEFCON | Makes quality/speed tradeoffs deliberate, not accidental |

All 8 proposals include exact protocol text ready to insert into AGENT-PROTOCOL.md. They are ordered by communication layer (how agents receive tasks → how they prepare → how they handle overload → how they track work → how they ship → how they adapt to pressure).

