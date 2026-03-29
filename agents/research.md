# RESEARCH — Protocol Improvement (Distributed Systems Lens)

---

## CURRENT TASK
- Task: Read AGENT-PROTOCOL.md carefully. Then read COORDINATION-RESEARCH.md for the patterns we researched. Your job: propose specific improvements to the Agent Protocol inspired by DISTRIBUTED SYSTEMS patterns (Raft consensus, Byzantine fault tolerance, circuit breakers, eventual consistency). For each improvement: (1) what pattern inspires it, (2) what specific rule or section to add to the protocol, (3) write it in the exact language that should go in the protocol doc. Be specific — write the actual text, not just descriptions. We want to make this the best agent orchestration protocol ever created. Write ALL proposals to OUTBOX.
- Status: Done

---

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
