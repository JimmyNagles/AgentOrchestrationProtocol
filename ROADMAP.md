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

---

## Codebase Map

> Once the project has code, map the folder structure here so all agents know where things go.

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
│   ├── claude-code.md
│   ├── codex.md
│   ├── gemini.md
│   └── cowork.md
├── designs/
└── [your code here]
```

---

## Phase 0: Ideation
> Define the idea before building. See `ideation/` folder.
> **Gate:** Phase 1 does not start until `ideation/HERO-MOMENT.md` has a clear hero moment.

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
| Agent | Task | Scope |
|-------|------|-------|
| Claude Code | — | — |
| Codex | — | — |
| Gemini | — | — |

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
1. Jimmy says "check status" to Cowork
2. Cowork reads all agent files, drains OUTBOXes, updates AGENT-STATUS.md
3. Cowork assigns next tasks from this roadmap into each agent file
4. Jimmy triggers each agent with their cold-start message

**Phase transitions:**
- Gemini QA must PASS before moving to next phase
- Jimmy reviews + pushes to GitHub before next phase starts
- Cowork updates this roadmap with any scope changes
