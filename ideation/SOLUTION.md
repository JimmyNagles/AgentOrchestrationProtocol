# SOLUTION DEFINITION
> What are we building? Keep it sharp. If you can't explain it simply, it's not ready.

---

## One-Liner

**AgentOffice** helps solo founders orchestrate AI coding agents by turning agent management into a pixel-art startup simulation where the work is real.

---

## Value Proposition

**What it does:** A gamified visual interface where you manage AI agents as pixel-art employees in your virtual office. You assign tasks, watch them work, track progress, and ship real code — all from a game-like UI instead of juggling terminal windows.

**Why it's better:** Every other tool treats agent orchestration as a developer utility. This treats it as an experience. You get visibility (see all agents at once), persistence (state survives across sessions via markdown files), coordination (handoffs and dependencies are visual), and motivation (XP, levels, streaks, leaderboards make shipping addictive).

**What makes it hard to copy:** The protocol layer (proven file-based state management) combined with the game layer (pixel art, progression mechanics, community leaderboards) creates a product that's both technically sound and emotionally engaging. Competitors would have to build both, and developer tool companies don't think in game mechanics.

---

## Core Features (MVP Only)

| Feature | What It Does | Why It's Essential |
|---------|-------------|-------------------|
| **Pixel Office View** | 2D pixel-art office with agent characters at desks. Agents animate based on real state (typing, reading, idle, blocked). | This IS the product — the visual layer that doesn't exist anywhere else |
| **One-Click Agent Launch** | Click an agent character → launches Claude Code (or other tool) with the right cold-start prompt and context | Eliminates copy-paste cold-start friction, the #1 daily annoyance |
| **Live State Sync** | Watches `.md` agent files → updates UI in real-time. Current task, status, outbox messages all visible at a glance | Solves "I can't see what my agents are doing" |
| **Task Board** | Assign tasks to agents from the UI. Writes to the agent's `.md` file. Shows dependency order. | Replaces manually editing markdown files for task assignment |
| **XP & Progress** | Agents earn XP for completed tasks. Level up visually. Streak tracking for daily shipping. | The engagement hook — transforms shipping from a chore into a game |

---

## What We're NOT Building (Yet)

- **Agent runtime** — we don't run agents ourselves, we orchestrate existing tools (Claude Code, Codex, etc.)
- **Code editor / IDE** — we're not replacing Cursor or VS Code, we sit alongside them
- **Mobile app** — web only for now
- **Team collaboration** — single founder first, multiplayer later
- **AI model hosting** — users bring their own API keys / subscriptions
- **Automated agent-to-agent communication** — the founder stays in the loop (for now)

---

## Agent Guide

> **For agents reading this:** This file is complete. Move to MARKET.md.
