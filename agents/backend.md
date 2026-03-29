# BACKEND — Protocol Improvement (Software Engineering Lens)

---

## CURRENT TASK
- Task: Read AGENT-PROTOCOL.md carefully. Then read COORDINATION-RESEARCH.md for the patterns we researched. Your job: propose specific improvements to the Agent Protocol inspired by SOFTWARE ENGINEERING practices (Linux kernel MAINTAINERS, code review processes, CI/CD pipelines, CODEOWNERS, incident response, pair programming). For each improvement: (1) what pattern inspires it, (2) what specific rule or section to add to the protocol, (3) write it in the exact language that should go in the protocol doc. Be specific — write the actual text, not just descriptions. Think about what makes open source projects with 1000+ contributors work smoothly. Write ALL proposals to OUTBOX.
- Status: Done

---

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
