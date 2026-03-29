# [PROJECT NAME] ROADMAP
> Goal: [What are we building and why?]
> Format: Phased — no dates, phases complete when they're done
> Owner: Cowork (Chief of Staff)
> Last updated: —

---

## Product Vision

> Write your product vision here. What does this product do? Who is it for? Why does it matter?

---

## Core Design Decisions

> Document key architectural and product decisions here as you make them.
> Format: Decision + Rationale
> Also logged in AGENT-STATUS.md DECISIONS LOG for cross-session persistence.

---

## Codebase Map

> Once the project has code, map the folder structure here so all agents know where things go.
> Include domain ownership — which agent owns which directories.

```
[project-name]/
├── AGENT-PROTOCOL.md
├── AGENT-STATUS.md
├── ROADMAP.md
├── SHARED.md
├── ideation/
│   ├── PROBLEM.md
│   ├── USER.md
│   ├── SOLUTION.md
│   ├── MARKET.md
│   ├── VALIDATION.md
│   └── HERO-MOMENT.md
├── agents/
│   ├── _template.md
│   └── [name].md (one per agent)
├── designs/
└── [your code here]
```

---

## Phase 0: Ideation
> Define the idea before building. See `ideation/` folder.
> **Gate:** Phase 1 does not start until ALL ideation files are complete.

**Status:** _Not started_

- [ ] PROBLEM.md — Problem defined
- [ ] USER.md — Target user defined
- [ ] SOLUTION.md — Solution and MVP scope defined
- [ ] MARKET.md — Competitive landscape mapped
- [ ] VALIDATION.md — Key assumptions identified
- [ ] HERO-MOMENT.md — Hero moment articulated

---

## What's Built

_Nothing yet._

---

## Phase 1: [Name]
> Description of what this phase delivers.

### What's Being Added
- ...

### Agent Assignments
| Agent | Task | Domain |
|-------|------|--------|
| [Agent] | — | — |

### Dependency Order
> Which agent needs to finish before the next can start?

```
[Agent A task] → [Agent B task] → [Agent C task]
```

---

## Phase 2: [Name]
> Not started.

---

## Phase 3: [Name]
> Not started.

---

## Future / Backlog

- ...

---

## How to Use This Roadmap

**Every session:**
1. Founder says "check status" to Cowork
2. Cowork reads all agent files, drains OUTBOXes, updates AGENT-STATUS.md
3. Cowork assigns next tasks from this roadmap into each agent file
4. Founder triggers each agent with their cold-start message

**Phase transitions:**
- QA must PASS before moving to next phase
- Founder reviews + pushes to GitHub before next phase starts
- Cowork updates this roadmap with any scope changes
