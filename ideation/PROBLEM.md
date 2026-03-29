# PROBLEM DEFINITION
> What problem are we solving? If you can't explain the pain, the product won't matter.

---

## The Problem

**Problem statement:** Solo founders and indie hackers who run multiple AI coding agents in parallel struggle to manage them because there's no visibility, no coordination, and no persistence — every session starts from zero.

**Who feels this pain?** Solo founders, indie hackers, and small-team developers who use 2-5 AI coding agents (Claude Code, Codex, Gemini, Cursor) to build products. They're technical enough to use CLI tools but overwhelmed by the orchestration overhead.

**How painful is it?** Daily frustration. They spend more time managing agents than building. Context is lost between sessions. Agents overwrite each other's work. Review backlogs pile up. Rate limits burn 3x faster with no visibility into spend. The cognitive overhead of juggling terminals often makes the whole setup slower than just coding alone.

---

## What Do People Do Today?

| Current Solution | What Works | What's Broken |
|-----------------|------------|---------------|
| Multiple terminal tabs/tmux panes | Simple, direct control | No visibility across agents, no state persistence, no coordination |
| Conductor (Melty Labs) | Centralized dashboard, git worktree isolation | Developer tool UX — no engagement layer, no motivation mechanics, macOS only |
| Markdown protocol files (like ours) | Persistence across sessions, clear roles | Manual copy-paste cold starts, no visual layer, founder is the message bus |
| Doing nothing (one agent at a time) | Simple | Leaves 80% of the speed gain on the table |

---

## Why Now?

1. **AI coding agents just hit mainstream.** Claude Code does 4% of all GitHub commits. Anthropic went from $1B to $19B ARR in 14 months. Cursor hit $2B ARR.
2. **Multi-agent is the new normal.** Gartner saw a 1,445% surge in multi-agent inquiries. Claude Code's Boris Cherny runs 10-20 agents in parallel.
3. **The tools exist but the experience doesn't.** Every competitor builds for developers-as-developers. Nobody is making this fun.
4. **"Vibe coding" created a new user.** 63% of vibe-coded apps are built by non-developers. These users need even more structure and guidance than power developers.
5. **Programmatic agent launching is now possible.** Claude Agent SDK, `claude -p`, Codex CLI all support non-interactive mode. You can launch agents from a UI.

---

## Agent Guide

> **For agents reading this:** This file is complete. Move to USER.md.
