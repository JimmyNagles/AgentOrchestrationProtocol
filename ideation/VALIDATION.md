# VALIDATION CHECKLIST
> What assumptions are we making? How do we test them before writing code?
> Every startup is a stack of assumptions. The fastest teams test them before building.

---

## Key Assumptions

| # | Assumption | Risk Level | How to Test | Result |
|---|-----------|------------|-------------|--------|
| 1 | Solo founders actually run 2+ agents in parallel regularly | Medium | Survey indie hackers + r/ClaudeAI. Count people discussing multi-agent workflows on Twitter/X. Check Claude Code usage patterns. | Evidence strong — Boris Cherny runs 10-20, Peter Steinberger ran 4-10, Simon Willison documented the workflow publicly |
| 2 | The coordination overhead is painful enough to pay for a solution | High | Interview 10 multi-agent users. Ask: "How much time do you spend managing agents vs. building?" | Reddit complaints confirm pain. Need direct interviews to confirm willingness to pay. |
| 3 | Gamification increases engagement over a plain dashboard | High | Build two prototypes — gamified vs. plain dashboard. A/B test with 20 users. Measure session length and return rate. | Pending — this is the riskiest assumption |
| 4 | People will pay $19-29/mo for this on top of existing AI subscriptions | High | Landing page with pricing → waitlist conversion. Target: 5%+ conversion. | Pending |
| 5 | File-based state (.md files) is sufficient for agent orchestration at scale | Medium | Stress test with 5+ agents, 50+ tasks, rapid file changes. Measure sync latency and conflicts. | Partially validated — the protocol has been used on ConciergeAI. Need scale testing. |
| 6 | The pixel art aesthetic appeals to the target audience (not seen as childish) | Medium | Show mockups to 20 target users. Ask: "Would you use this daily?" | Pixel Agents got featured in Fast Company and was well-received. Gather.town uses pixel art for serious work. Signal is positive. |
| 7 | Claude Code (and other agents) can be reliably launched from a web UI | Low | Build a proof-of-concept: Express backend → spawn `claude -p` → stream output to frontend | Claude Agent SDK exists. CLI supports `--output-format stream-json`. Technically validated. |

---

## Validation Methods

- [x] **Competitor analysis** — done. Competitors exist but none are gamified. Market is growing fast.
- [ ] **Founder interviews** — talk to 10 solo founders who use multi-agent workflows. Confirm pain and willingness to pay.
- [ ] **Landing page test** — put up a page with the game concept, pricing, and a waitlist. Target: 500 signups in 2 weeks.
- [ ] **Prototype test** — build the pixel office with live file sync. Show it to 20 users. Measure reaction.
- [ ] **Pre-sell** — can we get 10 people to pay $19/mo before the full product is built?

---

## Go / No-Go Signals

**Green light (build it):**
- 10+ founders say "I'd pay for this" in interviews
- Landing page converts at 5%+ to waitlist
- Prototype gets a "wow" reaction — people want to show it to friends
- At least 3 people pre-pay

**Red flag (rethink it):**
- Founders say "I'd use it if it were free" but won't pay — means the pain isn't acute enough
- The gamification feels gimmicky to the target audience — means the game layer needs rethinking
- File-based sync has >2s latency or causes conflicts — means the architecture needs work
- Nobody shares it — means it's not remarkable enough

---

## Agent Guide

> **For agents reading this:** This file is complete. Move to HERO-MOMENT.md.
> **This file is a hard gate.** Phase 1 does not start until key assumptions are identified.
