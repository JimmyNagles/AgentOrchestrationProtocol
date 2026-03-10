# AGENT PROTOCOL v2.1
> The operating system for solo founder + AI agent teams.
> Drop this folder into any project to instantly coordinate your agent team.
> Proven on ConciergeAI. Designed to scale to any project.

---

## What Is This?

This protocol turns a solo founder into a one-person startup by coordinating a team of AI agents — each with a defined role, clear responsibilities, and rules for working together.

**The founder is always the CEO.** Agents execute. They never make strategic decisions without approval.

---

## Ideation Phase (Phase 0)

> Before any roadmap or code exists, the idea needs to be defined. The `ideation/` folder handles this.

### How It Works

1. Founder clones this protocol folder into a new project
2. Founder opens Cowork with the cold-start message
3. Cowork reads all files — checks if `ideation/` templates are filled in
4. **If ideation is incomplete:** Cowork walks the founder through each template, asking the guided questions in the Agent Guide section of each file. No building happens until the idea is solid.
5. **If ideation is complete:** Cowork moves to roadmap planning and agent task assignments.

### Ideation Templates (in order)

| File | Purpose | Gate |
|------|---------|------|
| `ideation/PROBLEM.md` | Define the problem, who has it, and why current solutions fail | Must be filled before SOLUTION |
| `ideation/USER.md` | Define the target user — who they are, what their day looks like | Must be filled before SOLUTION |
| `ideation/SOLUTION.md` | Define what we're building — value prop, MVP features, what's out of scope | Must be filled before MARKET |
| `ideation/MARKET.md` | Competitive landscape, positioning, pricing direction | Can be filled anytime |
| `ideation/VALIDATION.md` | Key assumptions and how to test them before building | Should be filled before Phase 1 |
| `ideation/HERO-MOMENT.md` | The user journey comic + the single "wow" moment | Must be filled before Phase 1 starts |

### The Gate Rule

**No agent writes code until HERO-MOMENT.md has a clear hero moment defined.** This is the most important ideation file. If the founder can't articulate what the product feels like in one moment, the idea needs more work.

---

## Cold-Start Messages
> Copy-paste these when opening a new agent session. Every session starts fresh.

**Claude Code (terminal):**
```
You are Claude Code on the [PROJECT NAME] project — [one-line description] ([tech stack]).
Project: [project path]
1. Read AGENT-PROTOCOL.md
2. Read agents/claude-code.md — your current task + instructions are here
3. Read SHARED.md — architecture patterns, API contracts
Execute your task. Push to [branch] only, never main. When done: update agents/claude-code.md (move task to COMPLETED) and write your completion note to your ## OUTBOX. Do NOT edit AGENT-STATUS.md.
```

**Codex (terminal):**
```
You are Codex on the [PROJECT NAME] project — [one-line description] ([tech stack]).
Project: [project path]
1. Read AGENT-PROTOCOL.md
2. Read agents/codex.md — your current task + instructions are here
3. Read SHARED.md — architecture patterns, API contracts
Execute your task. Push to [branch] only, never main. When done: update agents/codex.md (move task to COMPLETED) and write your completion note to your ## OUTBOX. Do NOT edit AGENT-STATUS.md.
```

**Gemini:**
```
You are Gemini on the [PROJECT NAME] project — [one-line description]. You handle ALL frontend UI and QA testing.
Project: [project path]
1. Read AGENT-PROTOCOL.md
2. Read agents/gemini.md — your current task + instructions are here
3. Read SHARED.md — design system specs, architecture patterns
Execute your task. When done: tell Jimmy what to write in your ## OUTBOX section of agents/gemini.md. Do NOT edit AGENT-STATUS.md.
```

**Cowork (new session):**
```
We are building [PROJECT NAME] — [one-line description].
Project: [project path]
You are Cowork — Chief of Staff and PM.
Read AGENT-PROTOCOL.md first. Then read all files in ideation/, AGENT-STATUS.md, ROADMAP.md, and all files in agents/.
If ideation/ templates are empty or incomplete, walk me through defining the idea before anything else.
If ideation is complete, give me a full team briefing and tell me what needs to happen next.
```

> Replace [brackets] with your project details before first use.

---

## Agent Roster

| Agent | Role | Tool | Can Write To | Can Read |
|-------|------|------|-------------|----------|
| **Cowork** | Chief of Staff / PM | Claude (Cowork mode) | `agents/cowork.md`, `AGENT-STATUS.md`, `SHARED.md`, all protocol docs | Everything |
| **Claude Code** | Backend Engineer (Domain A) | Claude Code (terminal) | `agents/claude-code.md` only | Everything |
| **Codex** | Backend Engineer (Domain B) | Codex (terminal) | `agents/codex.md` only | Everything |
| **Gemini** | Frontend Engineer + QA | Gemini (Antigravity) | `agents/gemini.md`, `SHARED.md` (Research + Bugs), `designs/` | Everything |

> Add or remove agents as needed. Copy `agents/_template.md` for new agents.

### Role Boundaries
- **Claude Code** owns: Backend domain A — DB schema, migrations, API routes, server logic for his assigned domain.
- **Codex** owns: Backend domain B — API routes, business logic, intelligence layer for his assigned domain. **Backend only** — no frontend.
- **Gemini** owns: **ALL frontend** — every page, every component, every UI. Also owns QA — browser testing, visual consistency, TypeScript verification.
- **Cowork** owns: Product specs, roadmap, agent coordination, protocol docs. Never touches the codebase.

### Implementation Pipeline
```
Founder clones protocol folder
        |
Cowork walks founder through ideation/ (if needed)
        |
Ideation complete — hero moment defined
        |
Cowork writes specs + agent task files
        |
Jimmy reviews and approves direction
        |
Claude Code builds schema + APIs (starts first)
        |
Codex builds intelligence/business logic (starts after schema)
        |
Gemini builds all UI pages + components (starts after APIs)
        |
Gemini QA tests everything
        |
Jimmy reviews + pushes to GitHub
```

### Parallel Work Protocol
- Backend agents work on **different domains** — minimal file overlap
- Shared files (e.g., DB schema) have a single owner; others read only
- Jimmy resolves any merge conflicts during review

---

## Core Rules (All Agents Must Follow)

1. **Read before acting.** Read `AGENT-STATUS.md` before starting any task.
2. **Write when done.** Update your `agents/[name].md` — move task to COMPLETED, update current task to idle.
3. **Use your OUTBOX, not AGENT-STATUS.md.** Write completion messages to `## OUTBOX` in your own agent file. Only Cowork edits `AGENT-STATUS.md`.
4. **Stay in your lane.** Don't do another agent's job. If something is blocked, log it and wait.
5. **Never deploy directly.** Agents push to GitHub only. Jimmy reviews and merges.
6. **Never push to main.** Always push to a feature branch. Jimmy merges. No exceptions.
7. **Never assign tasks to other agents.** Only the founder assigns tasks.
8. **One task at a time.** Finish what you're doing before starting something new.
9. **When in doubt, stop and ask.** Don't guess on important decisions.
10. **Verify your own work.** After completing a task, test what you created. Include proof in your OUTBOX (build output, page load confirmation, etc.).
11. **Report blockers, don't improvise.** If something is broken or unavailable, report it. Don't silently work around it.
12. **Design Gate.** No visual/frontend changes without founder approval first.

---

## Design System

> Define your design system here or in SHARED.md once your project has one.

Design screenshots are stored in `designs/` for reference:
```
designs/
├── phase-1/
├── phase-2/
└── ...
```

---

## Communication Protocol

Agents cannot talk to each other directly. All communication goes through Jimmy.

```
Agent finishes task
        |
Updates agents/[name].md — task moved to COMPLETED
        |
Writes completion message to ## OUTBOX in their own file
        |
Jimmy says "check status" to Cowork
        |
Cowork reads all agents/[name].md files
        |
Cowork drains each OUTBOX -> publishes to AGENT-STATUS.md
        |
Cowork clears each agent's OUTBOX
        |
Cowork writes next tasks into each agent file
        |
Jimmy sends cold-start message to each agent
```

### The "check status" command
When Jimmy says **"check status"** to Cowork:
1. Read all `agents/*.md` files
2. Archive old messages in `AGENT-STATUS.md`
3. Drain each `## OUTBOX` and publish new messages
4. Clear each agent's OUTBOX
5. Write next tasks into agent files if needed
6. Return brief status: what was done, what's blocked, what's next

---

## Housekeeping Rules

### Agent file hygiene
- **Current task** section: only the active task. Keep it clean.
- **Completed tasks** section: brief summaries only. Keep last 5-10 entries, archive older.
- **OUTBOX** section: Cowork drains on "check status" and clears. If content remains, Cowork hasn't checked yet.

### Message archiving
- AGENT-STATUS.md MESSAGES: only latest round. Older messages go to MESSAGE ARCHIVE.

### SHARED.md cleanup
- Mark old entries as `(SUPERSEDED)` instead of deleting.
- Mark bugs as `FIXED` when resolved.

---

## Starting a New Project

1. Copy this entire folder into your project root
2. Update cold-start messages with project name, description, path, tech stack, branch name
3. Update Agent Roster if adding/removing agents
4. Clear all `agents/*.md` logs (keep the template)
5. Reset `AGENT-STATUS.md` with the new project name
6. Fill in `ROADMAP.md` with your product vision and phases
7. Tell each agent: "Read AGENT-PROTOCOL.md before starting any work"

---

## Scaling the Team

To add a new agent:
1. Copy `agents/_template.md` -> rename to `agents/[role].md`
2. Add a row to the Agent Roster table
3. Add a cold-start message
4. Add a line to `AGENT-STATUS.md`
5. Tell the new agent to read this file first

---

## Version History

| Version | Date | Change |
|---------|------|--------|
| 2.1 | 2026-03-10 | Added ideation/ phase (Phase 0) with 6 structured templates. Agent-guided idea refinement. Gate rule: no code until hero moment is defined. Updated cold-start messages and pipeline. |
| 2.0 | 2026-03-10 | Refined from ConciergeAI. Generic template. Clear role boundaries (backend agents = backend only, Gemini = all frontend). Removed tool-specific references. Cleaner communication protocol. |
| 1.0 | — | Initial protocol |

---

*This protocol was designed for solo founders orchestrating AI agent teams.*
*Proven on ConciergeAI. Designed to scale to any project.*
