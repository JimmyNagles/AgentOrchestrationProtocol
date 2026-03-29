# PRODUCT — Protocol Improvement (Biological Systems Lens)

---

## CURRENT TASK
- Task: Read AGENT-PROTOCOL.md carefully. Then read COORDINATION-RESEARCH.md for the patterns we researched. Your job: propose specific improvements to the Agent Protocol inspired by BIOLOGICAL SYSTEMS (ant colony stigmergy, bee swarm quorum sensing, immune system self-tolerance, neural specialization). For each improvement: (1) what pattern inspires it, (2) what specific rule or section to add to the protocol, (3) write it in the exact language that should go in the protocol doc. Be specific — write the actual text, not just descriptions. Focus on how agents can self-organize, self-correct, and coordinate without heavy central control. Write ALL proposals to OUTBOX.
- Status: Done

---

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
