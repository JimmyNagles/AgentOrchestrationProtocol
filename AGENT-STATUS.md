# AGENT STATUS DASHBOARD
> Project: AgentOffice
> Orchestrator: Main Claude Code session (Jimmy's terminal)
> Last updated: —

---

## LIVE STATUS

| Agent | Role | Status | Current Task | Last Update |
|-------|------|--------|-------------|-------------|
| Product | Product Manager | Idle | — | — |
| Backend | Backend Engineer | Idle | — | — |
| Frontend | Frontend Engineer | Idle | — | — |
| Designer | UI/UX Designer | Idle | — | — |

---

## MESSAGES
_No messages yet._

---

## RULES
- All agents are Claude Code instances launched with `claude -p --dangerously-skip-permissions`
- Orchestrator (main session) writes tasks to agent .md files and launches agents
- Agents update their own .md file when done (Status + OUTBOX)
- Agents ONLY modify files in their DOMAIN OWNERSHIP
- Founder reviews and pushes to GitHub

## BLOCKED
_Nothing blocked._

---

## DONE
_No completed work yet._

---

## DECISIONS LOG

| Date | Decision | Made by | Reason |
|------|----------|---------|--------|
| 2026-03-29 | All agents are Claude Code instances | Jimmy | Simplifies orchestration — one CLI, one launch pattern |
| 2026-03-29 | Orchestrator launches agents via `claude -p --dangerously-skip-permissions` | Jimmy | Agents need autonomous file/terminal access |

---

## MESSAGE ARCHIVE
_No archived messages._
