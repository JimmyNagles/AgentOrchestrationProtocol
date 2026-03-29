# DESIGNER — Protocol Improvement (Human Organizations Lens)

---

## CURRENT TASK
- Task: Read AGENT-PROTOCOL.md carefully. Then read COORDINATION-RESEARCH.md for the patterns we researched. Your job: propose specific improvements to the Agent Protocol inspired by HIGH-PERFORMANCE HUMAN TEAMS (ER closed-loop communication, film production call sheets, kitchen brigade system, NASA go/no-go polling, orchestra conductors). For each improvement: (1) what pattern inspires it, (2) what specific rule or section to add to the protocol, (3) write it in the exact language that should go in the protocol doc. Be specific — write the actual text, not just descriptions. Focus on communication clarity, error prevention, and coordination under pressure. Write ALL proposals to OUTBOX.
- Status: Done

---

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
