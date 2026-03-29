================================================================
COORDINATION PATTERNS RESEARCH — 4 Agent Reports
Date: 2026-03-29
================================================================

================================================================
AGENT: RESEARCH
================================================================
## OUTBOX

### RESEARCH COMPLETE: Real-World Coordination Patterns for AI Agent Orchestration

Full research documents saved to `research/DISTRIBUTED-SYSTEMS-PATTERNS.md`, `ideation/COORDINATION-PATTERNS.md`, and `research/AVIATION-PATTERNS.md`. Below is the complete synthesis.

---

## PART 1: DISTRIBUTED SYSTEMS

### 1.1 Raft Consensus
**How it works:** Independent nodes elect a leader via randomized timeouts. Leader replicates a log to followers. Entries committed when majority acknowledges. Term numbers (logical clocks) immediately dethrone stale leaders.

**Key rules:**
- **Quorum = floor(N/2) + 1.** Any two majorities overlap by at least one node — prevents split-brain.
- **One vote per term.** Each node votes once per term (first-come-first-served). At most one leader per term.
- **Election restriction.** Candidates must have the most up-to-date log to win. New leaders never lose committed data.
- **Heartbeats << election timeout.** Leader sends heartbeats every 50-150ms; election timeout is 150-300ms (randomized). Prevents unnecessary elections.
- **Commit only own-term entries.** A leader commits previous-term entries indirectly, by committing a current-term entry after them.

**Agent orchestration insight:** Map to coordinator election. If the coordinator agent crashes (stops heartbeating), another agent calls an election. Term numbers reject stale instructions from former coordinators. Log replication = shared task log committed only when majority of agents acknowledge.

### 1.2 Paxos Consensus
**How it works:** Two-phase protocol — Prepare (proposer gets promises from majority) then Accept (proposer sends value, acceptors accept if no higher prepare seen).

**Critical rule:** If any acceptor already accepted a value, the proposer MUST propose THAT value, not its own. This is what makes Paxos converge — once a value is chosen, all future proposals converge on it.

**Agent orchestration insight:** Any agent can propose "I'll handle task X." If a majority agree, assignment is locked. The "adopt previously accepted value" rule prevents assignment thrashing.

### 1.3 Byzantine Fault Tolerance (PBFT)
**The 3f+1 rule:** To tolerate f malicious/faulty nodes, you need 3f+1 total. Math: n-2f > f (honest responders must outnumber liars after removing non-responders). Quorum = 2f+1.

**Three phases:** Pre-Prepare (leader assigns sequence number) → Prepare (2f+1 votes confirm ordering) → Commit (2f+1 commits make it durable across view changes).

**Agent orchestration insight:** To tolerate 1 unreliable agent, you need 4 total. To tolerate 2, you need 7. For critical decisions, run 3+ independent agents and require 2/3 supermajority. View changes = replacing a malfunctioning coordinator by vote.

### 1.4 Blockchain / Proof of Stake
**Key mechanisms:**
- **PoW:** Brute-force hash search + longest chain rule. 6-confirmation rule makes reversal negligibly probable. 51% attack threshold.
- **PoS (Casper FFG):** Validators stake collateral. 2/3 supermajority finalizes checkpoints. Three slashable offenses (double proposal, surround vote, double vote). Correlation penalty scales with number of simultaneous offenders.

**Agent orchestration insight:** Reputation staking — agents earn trust over time, lose it on bad outputs. Correlation penalty is powerful: if multiple agents from the same model/provider all fail simultaneously, amplify the penalty (detecting correlated failures). "Proof of computation" — agents must show verifiable intermediate results, not just final answers.

### 1.5 Eventual Consistency / CRDTs
**Vector clocks:** Each node maintains per-node counters. Increment on local event, attach on send, pairwise-max on receive. If neither VC_a < VC_b nor VC_b < VC_a, events are concurrent (conflict!).

**CRDTs:** Data structures whose merge is commutative + associative + idempotent = guaranteed convergence without coordination.
- G-Counter: array of per-node integers, merge via pairwise max
- PN-Counter: two G-Counters (positive, negative)
- LWW-Register: (value, timestamp) pair, higher timestamp wins
- OR-Set: (element, unique_tag) pairs, union on merge

**Agent orchestration insight:** Use CRDTs for shared agent state — G-Counter for "total tasks completed," OR-Set for "active tasks." Vector clocks track causal dependencies between agent outputs. For task assignments use multi-value conflict resolution (surface to human). For metrics use automatic merge.

### 1.6 Circuit Breakers & Bulkheads
**Circuit breaker:** Three states — CLOSED (normal, tracking failures) → OPEN (all requests rejected, fail-fast) → HALF-OPEN (limited test requests). Transition thresholds: 50% failure rate or 5 absolute failures → open. 3 consecutive successes in half-open → close.

**Bulkhead:** Partition resources into isolated pools per dependency. One slow service can only exhaust its own pool — others unaffected.

**Full resilience stack:** Bulkhead → Circuit Breaker → Retry (exponential backoff + jitter) → Timeout → Call.

**Agent orchestration insight:** If an agent starts producing errors, trip its circuit breaker — stop sending tasks, wait for cooldown, send a "canary task" (known correct answer) to test recovery. Separate resource pools per agent type prevents one broken agent from blocking the pipeline.

---

## PART 2: MILITARY & NASA

### 2.1 CAPCOM — Single Point of Communication
**Rule:** ALL communication between Mission Control (~20 controllers + back rooms) and astronauts passes through ONE person (CAPCOM, always a fellow astronaut). Only exception: Flight Surgeon or Flight Director in emergencies.

**Why:** Prevents information overload on crew. Forces internal consensus before communicating. No contradictory instructions.

**Agent rule: One agent, one commander. Never multi-source an executing agent. Internal disagreement must be resolved BEFORE instructions reach the executor.**

### 2.2 Go/No-Go Polling
**Rule:** Before any irreversible event, Flight Director polls EVERY controller in fixed order. Each gives unambiguous binary: GO or NO-GO. Single NO-GO halts everything. No "maybe," no "probably fine."

**Why:** Eliminates ambiguity. Prevents bystander effect. Forces commitment. Creates shared awareness.

**Agent rule: Before irreversible operations, poll all validators. Each returns boolean, not confidence score. Any NO-GO blocks. Deterministic polling order so no validator is skipped. Log every vote with rationale.**

### 2.3 Flight Rules — Pre-Written Decision Trees
**Rule:** Hundreds of pages of IF/THEN rules written BEFORE the mission, in calm conditions, by the same controllers who will execute them. When a situation matches a rule, execute it without debate. Improvise only when no rule exists.

**Why:** Removes decision latency. Prevents emotional/political pressure from overriding technical judgment. Separates "policy time" from "execution time."

**Agent rule: Pre-define decision rules for known failure modes. If situation matches a rule, agent executes without escalation. Escalation is for novel situations only. Store rules separately from agent logic for auditability.**

### 2.4 Apollo 13 Crisis Pattern
**What Kranz did:** (1) Immediately reframed mission objective ("this is now a survival mission"). (2) Assigned parallel workstreams to independent problems. (3) Built on pre-existing partial procedures. (4) Flight Director had absolute authority.

**Agent rule: When a task fails, immediately re-scope — do not pursue original goal with broken assumptions. Orchestrator has absolute authority during error recovery. Crisis procedures must be parallelizable. Pre-build partial procedures for likely failures.**

### 2.5 OODA Loop (Boyd)
**Rule:** Observe → Orient → Decide → Act. Victory goes to whoever cycles faster. Boyd's key insight: Orient is the competitive advantage — it's where mental models, context, and experience live. Speed without quality orientation is dangerous.

**Agent rule: Every agent task should have explicit O-O-D-A phases. Orient is mandatory — raw data in, action out = dangerous. The feedback loop (Act → Observe) must be closed. Agents that act without observing results diverge from reality.**

### 2.6 Auftragstaktik — Mission-Type Orders
**Rule:** Tell subordinates WHAT to achieve and WHY. Never HOW. Subordinate has full execution freedom. Can deviate from plan if deviation still serves commander's intent.

**Agent rule: Task assignments specify desired outcome and constraints, NOT step-by-step instructions. Over-specifying HOW creates fragility. If step 3 of 10 fails, agent is stuck. If agent only knows the goal, it can find another path. This is declarative vs. imperative tasking.**

### 2.7 Commander's Intent
**Rule:** Every order contains a plain-language statement of purpose and desired end state. When communications fail, plans fall apart, and situation changes — everyone falls back to commander's intent. Must be short enough to remember under extreme stress.

**Agent rule: Every task includes intent statement separate from instructions. Format: "The purpose is [WHY]. The desired end state is [WHAT SUCCESS LOOKS LIKE]." If agent cannot complete specific instructions, it falls back to intent and finds alternative path. Intent must fit in a system prompt.**

### 2.8 Rules of Engagement (Graduated Response)
**Rule:** Define circumstances, conditions, and degree of response. Enforce escalation steps: (1) Retry silently (2) Log and retry with backoff (3) Alert orchestrator, continue (4) Alert orchestrator, pause (5) Halt all operations.

**Agent rule: Define explicit ROE for agents — what they can do autonomously vs. what requires escalation. Actions must be proportional. Require "positive identification" of problems before corrective action. Every agent carries a simplified ROE card (its rules in its system prompt).**

### 2.9 Two-Person Rule (Nuclear)
**Rule:** No single person may have unsupervised access to nuclear weapons. Two cleared, task-knowledgeable individuals must always be present. Physical constraints enforce the rule (Minuteman: two locks, separated key slots, both keys turned simultaneously).

**Agent rule: Irreversible or high-impact operations require approval from two INDEPENDENT agents (not the same agent twice). They must have independent context. Design physical constraints (API-level enforcement), not just policy constraints (prompts).**

### 2.10 Challenge-Response Authentication
**Rule:** President's "biscuit" (daily auth codes) + NMCC challenge = mutual authentication. Codes rotate daily. Both sides prove identity before nuclear orders transmit.

**Agent rule: Before executing high-impact instructions, agent verifies source is authorized. Use rotating tokens, not static credentials. Mutual verification — orchestrator proves identity to agent AND agent proves it received correct instruction. Prevents "confused deputy" attacks.**

### 2.11 Confirmation Brief / Brief-Back
**Rule:** Two steps: (1) After receiving orders, subordinate restates understanding. Commander corrects. (2) After planning, subordinate presents plan. Commander verifies it serves intent. Both happen BEFORE execution.

**Agent rule: After receiving task, agent echoes back understanding in its own representation. Orchestrator validates semantic correctness (not just acknowledgment). After planning but before execution, agent's plan is reviewed against original intent. Catches misunderstandings at cheapest possible moment.**

### 2.12 Positive Control (PAL)
**Rule:** Default state is SAFE. Affirmative, authenticated action required to arm/launch. Loss of communication = weapon stays safe. Silence means "do nothing," never "proceed."

**Agent rule: Default agent state is IDLE, not EXECUTING. Loss of communication with orchestrator = pause, not continue. Dangerous operations require POSITIVE authorization, not just absence of prohibition. Never design a system where silence means consent.**

### 2.13 Triple Modular Redundancy (TMR) with Voting
**Rule:** Three independent systems compute same result. Majority vote wins. If one fails, other two outvote it. Used in Saturn V (even voters were triplicated).

**Agent rule: For critical decisions, run three TRULY independent agents (different models, different prompts, different approaches). Three copies of same agent = no protection against systematic errors. Voter must be simpler and more reliable than what it judges. If all three disagree, escalate to human.**

### 2.14 Space Shuttle: Quad Redundancy + Diverse Backup
**Rule:** Four identical computers run in lockstep (PASS by IBM) + one backup running DIFFERENT software by a DIFFERENT company (BFS by Rockwell). Quad handles hardware failures. Diverse backup handles software bugs that would crash all four simultaneously.

**Agent rule: Redundant agents with same model protect against stochastic errors but NOT systematic biases. For systematic error protection, use a DIVERSE backup — different model, different prompt, different architecture. Backup should be simpler than primary. Switchover must be fast (single action).**

### 2.15 Independent Verification & Validation (IV&V)
**Rule:** Separate organization, no reporting relationship to dev team, independently verifies software. Mandated after Challenger. 4-7 testing levels, each by different people. IV&V defines their own test criteria.

**Agent rule: Agent that generates plan must NOT be only validator. Validator must be organizationally independent — different system prompt, different context, different optimization target. Multiple validation levels with different perspectives. Validators define their own test criteria.**

### 2.16 ICS Span of Control: 3-7, Optimal 5
**Rule:** No supervisor has fewer than 3 or more than 7 direct reports. Optimal = 5. If >7, split into sub-units with intermediate supervisors. Applies at EVERY level.

**Scaling formula:** N agents requires ceil(log₅(N)) orchestration layers. 5 agents = 1 layer. 25 = 2 layers. 125 = 3 layers.

**Agent rule: No orchestrator manages more than 7 sub-agents directly. When pool exceeds 7, create intermediate orchestrators (team leads). This prevents orchestrator overload — tracking 20 agents = dropping state.**

### 2.17 Modular Organization (ICS)
**Rule:** Only activate parts of the org that the incident requires. Start small, expand as needed, contract when need passes.

**Agent rule: Start with minimal agents. Add only when span-of-control limits are hit or new capabilities needed. Have full org template defined but only instantiate needed parts. Deactivate agents when function is no longer needed.**

### 2.18 Transfer of Command (ICS)
**Rule:** When command transfers, full briefing required (situation, plan, issues, resources). All personnel notified. Discrete transfer — never ambiguous authority.

**Agent rule: When swapping orchestrator agents, full state must be serialized and transferred. All sub-agents notified of new orchestrator. Never a moment with two orchestrators or zero orchestrators. State transfer must be verified (incoming echoes back understanding).**

### 2.19 Incident Action Plan (IAP)
**Rule:** Written plan for each operational period. Contains objectives, org chart, assignments, comms plan. Everyone works from same document. Refreshed each period.

**Agent rule: Maintain shared written plan all agents can reference. Plan has TTL — must be refreshed periodically. When plan updates, ALL agents receive update. Plan is single source of truth — if agent's local state conflicts, plan wins.**

---

## PART 3: AVIATION

### 3.1 Separation Minima
**Rule:** Hard non-negotiable buffers: 3nm radar separation within 40nm of antenna, 5nm beyond. 1,000ft vertical below FL290, 2,000ft above. Numbers scale with uncertainty — less accurate position data requires larger buffers.

**Agent rule: Agents on shared resources need defined "separation minima" — minimum time or state gaps between conflicting operations. Less observable agent state = wider buffer. Hard resource locks with uncertainty-proportional timeouts.**

### 3.2 Sector Handoff Protocol (FAA 7110.65)
**Rule:** 5-step sequence: (1) Transferring controller resolves all conflicts first. (2) Completes electronic handoff BEFORE aircraft enters new sector. (3) Issues necessary restrictions. (4) Receiving controller verifies position. (5) Neither alters aircraft during handoff without verbal approval from other.

**Agent rule: Prepare state → transfer metadata → receiving agent acknowledges → transferring agent releases. Never a gap in ownership. Transferring agent must "resolve conflicts" in own domain — no passing dirty state. Handoff is explicit acknowledged event, not implicit.**

### 3.3 Readback/Hearback (Closed-Loop Communication)
**Rule:** Three steps: (1) Controller issues clearance. (2) Pilot reads back safety-critical elements (altitude, heading, speed, runway — NOT just "roger"). (3) Controller verifies readback or corrects. Controller silence is NOT confirmation.

**Agent rule: Every critical instruction: orchestrator issues → agent echoes back specific parameters → orchestrator explicitly confirms or corrects. "Acknowledged" proves nothing about comprehension. Agent must echo THE SPECIFIC PARAMETERS. Implement explicit ACK/NACK. Timeout on missing confirmation = restart loop.**

### 3.4 PACE Escalation (CRM)
**Rule:** Junior crew escalates through four graded steps: Probe (ask question) → Alert (state concern) → Challenge (propose action) → Emergency (take control). Created after Tenerife disaster (1977, 583 dead) where steep authority gradient suppressed dissent.

**Agent rule: Sub-agents need structured escalation protocol: Level 1 log observation → Level 2 flag explicitly → Level 3 propose alternative → Level 4 refuse instruction and escalate to human. No "infallible orchestrator" — every agent can halt unsafe operations regardless of hierarchy.**

### 3.5 Sterile Cockpit Rule (14 CFR 121.542)
**Rule:** During critical phases (below 10,000 feet), NO non-essential conversation, duties, or distractions. Binary rule, not a suggestion. Enacted after 1974 Eastern Airlines crash where crew was distracted.

**Agent rule: Define "critical phases" for agent operations (deployment, data migration, financial transactions). During critical phases: no background tasks, no speculative operations, no optimization side-quests. Implement `sterile_mode` flag that restricts capabilities.**

### 3.6 Cross-Checking and Monitoring (FAA AC 120-51E)
**Rule:** Pilot Monitoring has explicit duties: monitor instruments continuously, call out deviations (altitude, heading, speed), manage radio, verify all Pilot Flying actions. Standard callouts at defined thresholds: "one thousand to go," "airspeed," "sink rate," "bank angle."

**Agent rule: Every executor agent should have a paired monitor with defined deviation thresholds that REQUIRE alerts. Monitor role is first-class, not optional overhead. Define numeric thresholds. Monitor must have INDEPENDENT access to ground truth — not just executor's self-reported status.**

### 3.7 Challenge-Response Checklists
**Rule:** PM reads challenge ("Landing gear?"), PF physically VERIFIES state and responds ("Down, three green"). If response is incorrect or missing, checklist STOPS. PF must verify actual state, not respond from memory.

**Agent rule: Pre-execution checklists where orchestrator challenges and executing agent must verify and respond with ACTUAL state. "Database backup exists?" → agent actually checks → "Backup verified: 2024-03-29T10:00:00Z, 4.2GB, checksum abc123." NOT: "Yes."**

### 3.8 Three-Tier Checklist System
**Rule:** Normal checklists = confirmatory (verify what was already done). Abnormal = read-and-do (prescriptive, one step at a time). Emergency = memory items first (3-5 immediate actions from training), then systematic verification with written checklist.

**Agent rule: Normal ops = post-action verification. Abnormal ops = step-by-step guided remediation. Emergency ops = pre-loaded instant responses (circuit breakers, kill switches) that execute WITHOUT orchestrator approval, followed by systematic verification. Agents need "memory items" — hardcoded immediate responses to critical failures.**

### 3.9 Surgical Checklist (Gawande/WHO)
**Key result:** 19-item checklist across 3 junctures (before anesthesia, before incision, before leaving OR). NEJM 2009: complications dropped 36%, deaths dropped 47%, across 8 countries. Takes ~2 minutes.

**Key insight:** Checklist doesn't add knowledge — it catches OMISSION errors. The "Time Out" forces entire team to pause and synchronize mental models. It flattens authority gradient — a nurse has institutional backing to halt a surgeon.

**Agent rule: "Time Outs" before irreversible operations — all participating agents pause, state understanding, confirm readiness. 2-minute investment for 47% error reduction = the ROI of pre-execution verification is enormous.**

### 3.10 Swiss Cheese Model (James Reason, 1990)
**Rule:** Accidents require failures in MULTIPLE defensive layers simultaneously. Four layers: Organizational influences → Unsafe supervision → Preconditions → Unsafe acts.

**Critical distinction:** Active failures (operator errors, immediate consequences) vs. Latent failures (organizational decisions made long before, lying dormant until they combine with active failures). **Latent failures are the greater danger because they are invisible to the front-line operator.**

**Agent rule: Design multiple independent defensive layers: (1) Input validation (2) Agent-level guardrails (3) Orchestrator review (4) Pre-execution verification (5) Runtime monitoring. NEVER remove a safety check because "it never catches anything." Audit for LATENT failures — model choice, prompt design, timeout settings, missing tests. Incident post-mortems must trace through all layers, not stop at the active failure.**

### 3.11 ICAO Standard Phraseology
**Rule:** Fixed vocabulary where each phrase has exactly ONE meaning: AFFIRM (yes), NEGATIVE (no), ROGER (received, NOT "I agree"), WILCO (will comply), UNABLE (cannot comply, requires reason), CORRECTION (error, correct version follows), DISREGARD (ignore last transmission). Numbers spoken digit-by-digit. No politeness hedging.

**Agent rule: Define fixed vocabulary for inter-agent communication: ACK, NACK, UNABLE(reason), WILCO, CONFIRM, CORRECTION, DISREGARD, HALT. No natural language for status communication between agents — use structured enumerated message types. Full identifiers (UUIDs, ISO timestamps), not abbreviations.**

### 3.12 ATIS — Versioned Shared State
**Rule:** ATIS broadcasts standardized airport info continuously with sequential letter identifier (Alpha, Bravo, Charlie...). Pilots report "I have information Bravo" — instant version check. New ATIS issued when conditions change.

**Agent rule: Shared context should be published ONCE (pub/sub), not passed individually. Use version identifiers. Agents report which version they have. "I have context version 7" → "Current is version 9, updating you."**

### 3.13 Automation Complacency
**Key data:** NASA ASRS: 77% of automation-overreliance incidents involved vigilance failure. Humans stop monitoring reliable systems.

**Case studies:** Eastern 401 (1972) — autopilot mode change unnoticed, 101 dead. Air France 447 (2009) — pilots couldn't fly manually when autopilot disconnected, 228 dead.

**Paradox:** The more reliable the automation, the MORE complacent the human becomes, the LESS prepared they are when it fails.

**Agent rule: Do NOT assume "human in the loop" means human is actually monitoring. Require periodic ACTIVE engagement at checkpoints (not passive observation). When agent enters degraded mode, FORCE attention — halt and require explicit acknowledgment. Regularly test supervisor agents with scenarios where automation produces subtly wrong results.**

### 3.14 Mode Confusion
**Data:** Average 2.01 mode confusion events per pilot per year. Common patterns: inadvertent mode change, silent mode reversion, ambiguous mode indication.

**Root cause:** Mental model mismatch — operator's model of system state ≠ system's actual state.

**Agent rule: Minimize distinct modes. When mode changes occur, announce LOUDLY and confirm. Make current mode inspectable at all times (primary status, not buried in logs). Never silently revert to a less capable mode — explicitly announce degradation.**

---

## SYNTHESIS: 10 UNIVERSAL PRINCIPLES ACROSS ALL THREE DOMAINS

| # | Principle | Distributed Systems | Military/NASA | Aviation |
|---|-----------|-------------------|---------------|----------|
| 1 | **Single point of command** | Leader election (Raft) | CAPCOM, Unity of Command | Pilot-in-command, single controller per sector |
| 2 | **Fail-safe defaults** | Circuit breaker (fail-fast) | Positive control (default=safe) | Go/No-Go (any NO-GO stops) |
| 3 | **Pre-computed decisions** | Flight rules (IF/THEN) | ROE, Commander's Intent | Emergency checklists, memory items |
| 4 | **Echo-back verification** | Two-phase commit | Brief-back | Readback/hearback |
| 5 | **Independent validation** | TMR voting, PBFT 3f+1 | Two-person rule, IV&V | Cross-checking, paired monitor |
| 6 | **Bounded span** | Bulkhead partitioning | ICS 3-7 span of control | Sector size limits |
| 7 | **Declare WHAT not HOW** | Eventual consistency (convergence guarantee) | Auftragstaktik | Clearances (destination, not route micro-management) |
| 8 | **Modular scaling** | Horizontal scaling, sharding | Modular ICS org | Sector splitting |
| 9 | **Diverse redundancy** | Different implementations (Shuttle BFS) | Multiple intel sources | Independent flight computers |
| 10 | **Closed feedback loops** | Heartbeats, anti-entropy | OODA loop | Closed-loop communication, ATIS versioning |

---

## TOP RECOMMENDATIONS FOR AGENT PROTOCOL

Based on all research, these are the highest-impact patterns to steal:

1. **Closed-loop communication** (readback/hearback) — Agent echoes back interpreted command with specific parameters before executing. Orchestrator explicitly confirms. Single highest-impact safety improvement.

2. **Go/No-Go polling** — Before irreversible operations, poll all validators. Binary answers. Any NO-GO blocks. No "probably fine."

3. **Commander's Intent** — Every task includes WHY and WHAT SUCCESS LOOKS LIKE, separate from HOW. Agents fall back to intent when specific instructions fail.

4. **Circuit breaker pattern** — If an agent starts failing, stop sending tasks. Cooldown. Canary task to test recovery. Prevents cascade.

5. **Swiss Cheese layered defense** — Multiple independent safety layers. Never remove one because "it never catches anything." Audit latent failures (prompts, configs, policies), not just runtime errors.

6. **Sterile mode** — During critical operations (deploy, migration), agents are restricted to the task at hand. No side-quests.

7. **Three-tier checklists** — Normal = verify after. Abnormal = guided remediation. Emergency = hardcoded instant response + verify after.

8. **Span of control (max 7)** — No orchestrator manages more than 7 agents. Create intermediate layers when scaling.

9. **Diverse redundancy** — For critical decisions, use different models/approaches, not copies of the same agent. Systematic errors beat identical redundancy.

10. **Positive control** — Default state is IDLE. Silence means STOP. Dangerous actions require explicit opt-in. Never design a system where silence means consent.

================================================================
AGENT: PRODUCT
================================================================
## OUTBOX

### Biological Systems Research — Patterns for AI Agent Orchestration

Research complete across four biological coordination systems: ant colonies, immune system, bee swarms, and biological neural networks. Below are the key mechanisms and how they map to AI agent orchestration.

---

## 1. ANT COLONIES

### Stigmergy — Indirect Coordination Through Environment
Ants coordinate by leaving pheromone traces in the environment rather than communicating directly. An ant returning from food deposits pheromone; others probabilistically follow stronger trails. Pheromone evaporates over time, so trails to depleted sources naturally fade. Multiple pheromone types serve different purposes (alarm, trail, recruitment). The trail network functions as a **shared external memory** — no individual ant comprehends the whole system.

**AI pattern:** Agents communicate through shared environmental state (task board, event log, shared files) rather than direct messaging. Communication overhead scales with environmental complexity, not agent population — giving near-constant scaling.

### Shortest Path Selection (Double Bridge Experiment)
Ants initially choose paths randomly. Ants on the shorter bridge complete round trips faster, depositing more pheromone per unit time. Positive feedback amplifies this advantage until ~95-100% of traffic converges on the shorter path.

**AI pattern:** Quality-weighted reinforcement. Better solutions naturally accumulate more signal through faster feedback cycles.

### Collective Decision-Making — Nest Site Selection (Temnothorax ants)
A multi-phase protocol: (1) Scouts independently explore and evaluate candidate sites on multiple criteria. (2) Quality-dependent delay — better sites get advertised sooner. (3) Tandem running — slow, one-on-one recruitment (teaching). (4) Quorum sensing — ants measure local population density through encounter rates. Threshold is proportional to colony size. (5) Transport — once quorum reached, switch from slow recruitment to rapid carrying (~3x faster).

**AI pattern:** Multi-phase evaluation with quorum-based commitment. Better results get priority through latency advantage. Commitment triggers a mode switch from deliberation to execution.

### Division of Labor — Response Threshold Model
Each task has a stimulus intensity that increases when unattended. Each ant has a response threshold per task type. Probability of engagement follows a sigmoid: P = s^n / (s^n + θ^n). Low-threshold ants engage first, reducing stimulus before high-threshold ants need to act. Different thresholds create emergent specialization without a dispatcher. Converges to near-optimal allocation in O(log N) time.

**AI pattern:** Agents with different activation thresholds for different task types. Tasks accumulate urgency when unattended. Specialization emerges naturally without a central scheduler.

### Interaction-Rate Regulation (Deborah Gordon's Harvester Ants)
Outgoing foragers decide whether to leave based on the **rate** of encounters with returning foragers. High contact rate = food plentiful = go forage. Low rate = stay. A closed-loop excitable system tracking resource availability in real time with zero global knowledge.

**AI pattern:** Agent decisions based on rate of incoming signals, not centralized state queries. Naturally load-balances.

### Key Ant Colony Mechanisms Summary
1. **Stigmergic coordination** — shared environment as communication medium
2. **Pheromone decay** — stale signals auto-expire, preventing lock-in
3. **Positive feedback + evaporation + stochastic exploration** — the three-part balance for convergence + adaptability
4. **Quality-dependent latency** — better solutions get advertised faster
5. **Quorum sensing** — collective commitment only after sufficient independent agreement
6. **Graceful degradation** — linear performance loss when agents are removed, not exponential

---

## 2. IMMUNE SYSTEM

### Decentralized Cell Coordination
No central controller. Coordination emerges from local interactions between specialized cell types via standardized molecular interfaces:
- **Dendritic cells (scouts):** Detect pathogens, capture antigens, migrate to lymph nodes, present structured context to T cells via 3 simultaneous signals: (1) antigen payload, (2) costimulatory confirmation "this is real," (3) cytokines encoding threat type
- **T Helper cells (coordinators):** Orchestrate the broader response. TH1 for intracellular pathogens, TH2 for extracellular
- **Cytotoxic T cells (assassins):** Kill infected cells by reading MHC Class I "status reports" that every cell displays
- **B cells (factories):** A single B cell produces ~4,000 plasma cells in 7 days, each secreting 2,000+ antibodies/second
- **Macrophages (cleanup/sentinels):** Engulf pathogens, present antigens, recruit others via cytokines

**AI pattern:** No agent has global knowledge. Each operates on local information via standardized interfaces. Scouts detect and classify; coordinators route; specialists execute; factories scale output.

### Self-Tolerance — Two-Layer Safety Architecture
**Central tolerance (development-time):** T cells in thymus undergo positive selection (can they bind MHC?) then negative selection (do they bind self too strongly? → deleted). Some self-reactive cells become Regulatory T cells (Tregs) instead of being destroyed. B cells in bone marrow: strong self-binders deleted, moderate self-binders silenced (anergy).

**Peripheral tolerance (runtime):** Tregs suppress effector T cells using IL-10, TGF-beta. They consume IL-2 (T cell growth signal), raising activation threshold — a quorum-sensing mechanism. T cells encountering antigen WITHOUT costimulatory signal become anergic — silenced. Two-signal requirement: antigen alone never triggers action.

**AI pattern:** Safety at two levels — validation before deployment (central) + runtime monitoring (peripheral). Dedicated suppressor agents maintain stability. Dual-confirmation required before destructive actions — a single signal is never enough.

### Two-Tier Response System (Innate + Adaptive)
**Innate (fast, generic):** Minutes to hours. Pattern Recognition Receptors detect conserved pathogen patterns. No memory, no specificity. Complement system "paints" threats with C3b protein for downstream processing.

**Adaptive (slow, precise, remembers):** 4-7 days for primary response. Generates antigen-specific receptors via gene rearrangement (billions of unique variants). Selected cells expand clonally. Creates memory for faster future responses.

**Bi-directional handoff:** Innate → Adaptive: DCs relay structured pathogen context. Adaptive → Innate: T helper cytokines recruit and direct innate effectors. Antibodies opsonize pathogens for innate phagocytosis.

**AI pattern:** Fast rule-based first responders handle known patterns. Slower reasoning agents handle novel threats. The fast system provides structured context to the slow system. After resolution, knowledge feeds back to improve fast responses.

### Cytokine Signaling — The Communication Protocol
Five categories: Interleukins (cell-to-cell), Interferons (antiviral alerts), TNFs (pro-inflammatory/cell death), Chemokines (directional GPS gradients), Colony Stimulating Factors (scale production).

Signaling scopes: Autocrine (self), Paracrine (local), Endocrine (global). Properties: Pleiotropy (same message, different meaning per cell type), Redundancy (multiple signals for same effect = fault tolerance), Synergy/Antagonism (signals amplify or cancel each other).

**Failure mode — Cytokine Storm:** Runaway positive feedback. Macrophages release signals that amplify inflammation, triggering more release. Without dampening (Tregs, IL-10, decoy receptors), the system destroys its own tissue.

**AI pattern:** Typed, structured messages with different scopes (local/regional/global). Same message can mean different things to different agent types. Redundancy provides fault tolerance. Quorum sensing prevents premature commitment. **Critical: runaway feedback loops require dedicated dampening mechanisms and circuit breakers.**

### Clonal Selection and Expansion — Auto-Scaling
Pre-existing diversity: millions of lymphocytes with unique receptors, ~1 in 100K-1M matches any given antigen. Selection: only matching cells activate. Expansion: exponential clonal division (specific T cells go from 1-in-100K to 1-in-100). Differentiation during expansion into effector cells (short-lived fighters), memory cells (long-lived reserves), and plasma cells (antibody factories). Contraction: most effectors die via apoptosis after threat eliminated.

**AI pattern:** Maintain diverse agent templates. Select matching template for task. Spin up many instances (horizontal scaling). Scale down after completion. Preserve "memory" instances with learned context.

### Memory Cells — Long-Term Learning
Multiple memory tiers: Central memory (lymph nodes, rapid proliferation), Effector memory (patrol peripheral tissues, immediate response), Tissue-resident memory (permanently stationed, fastest local response), Stem cell memory (self-renewing, can regenerate all subtypes).

Affinity maturation: Memory B cells undergo mutation + selection in germinal centers, producing better antibodies than initial response. Secondary responses are faster, larger, and higher affinity.

**AI pattern:** Persist solution patterns at multiple tiers: central knowledge base, edge caches, local agent context. Each re-encounter produces a faster, more refined response. Self-renewing templates can regenerate specialized instances.

---

## 3. BEE SWARMS

### Nest-Site Selection — Parallel Independent Evaluation
When a colony swarms (~10,000 bees), several hundred scouts independently search for nest cavities. Each scout spends ~40 minutes evaluating a site across multiple criteria (volume ~40L preferred, entrance ~12.5 cm², height ~3m, south-facing, dry). Reports back via waggle dance with vigor proportional to site quality: ~100 waggle circuits for first-rate vs ~12 for mediocre.

**AI pattern:** Parallel, independent evaluation by autonomous agents. Same quality rubric, different regions of solution space. Better results get more "airtime."

### Waggle Dance — Compact Structured Communication
A single physical signal encodes three variables: Direction (angle relative to vertical = angle to sun), Distance (waggle duration, measured via optic flow), Quality (dance tempo — higher quality = more waggle runs per minute). Higher-quality signals get more bandwidth through repetition.

**AI pattern:** Compact structured message format encoding location, cost, and confidence score. Better results get amplified through repetition.

### Swarm Intelligence — The Six Core Mechanisms
1. **Decentralization:** No boss bee. Queen plays zero role in site selection.
2. **Positive feedback:** Better sites attract more dances → more recruits → more dances (gradient ascent).
3. **Negative feedback:** Dance vigor naturally decays over time. Prevents lock-in to early discoveries.
4. **Cross-inhibition via stop signals:** Scouts head-butt dancers advertising competing sites with a 380ms buzz, causing them to shorten/cease dancing. Targeted suppression of competing alternatives.
5. **Massively parallel processing:** Hundreds of scouts explore simultaneously.
6. **Independent evaluation:** Each recruited scout performs her own firsthand assessment — no hearsay.

**AI pattern:** The three-mechanism model (amplification + cross-inhibition + decay) is a complete consensus algorithm. Handles both clear winners and deadlocks without global knowledge.

### Quorum Sensing — Commitment Threshold
Scouts monitor how many other scouts are simultaneously present at a site. Threshold: ~10-20 scouts (Seeley found ~15 on Appledore Island). Once quorum reached, scouts switch from dancing to "piping" — a signal telling the swarm to warm flight muscles to 35°C for takeoff. Piping spreads through the cluster over 30-60 minutes before liftoff.

Speed-accuracy tradeoff: 2 sites → fast decision. 5 equally good sites → nearly twice as long. Quorum threshold balances speed vs reliability.

**AI pattern:** A quorum is NOT a majority vote — it's a threshold of independently-arriving confirmations. Require N independent agents to converge on the same solution before committing. Two-phase commit: first reach quorum, then broadcast "prepare to execute."

### Brain Analogy (Seeley et al., Science 2011)
The bee swarm decision process is structurally identical to competing neural integrator circuits in vertebrate brains: evidence accumulates in competing populations, cross-inhibition sharpens competition, a threshold triggers commitment.

### Seeley's Five Principles of Effective Group Decision-Making
1. Compose the group of individuals with shared interests
2. Minimize the leader's influence on group thinking
3. Seek diverse solutions (scouts explore independently without bias)
4. Update through open debate (waggle dances are public broadcasts)
5. Use quorum response for dependable resolution (not unanimity, not majority vote)

---

## 4. BIOLOGICAL NEURAL NETWORKS

### Synaptic Signaling — Dual Communication Channels
**Chemical synapses (primary):** Unidirectional, asynchronous, 0.5-5ms latency. Neurotransmitter release converts electrical to chemical signals. NMDA receptors act as coincidence detectors (AND-gates). Retrograde signaling: postsynaptic neurons send endocannabinoids back, creating bidirectional feedback.

**Electrical synapses (gap junctions):** Bidirectional, faster, synchronize groups to fire in unison.

**Mixed synapses:** Both at same site — fast component (electrical, sub-ms) + slow component (chemical, ms-seconds).

**AI pattern:** Dual communication channels: fast sync (shared memory/events) for coordination + slower rich messaging (structured API calls) for nuanced delegation. Downstream agents send feedback to upstream about load, confidence, errors.

### Functional Specialization — Goal-Based Delegation
Prefrontal cortex implements rostro-caudal hierarchy: caudal PFC (concrete, reactive) → mid-lateral PFC (rule-based routing) → rostral PFC (abstract goals). PFC works through **goal-based biasing** — maintains task goals that bias signals through other structures. Sets the "what" and "why"; downstream regions implement the "how."

**Adaptive coding:** Same neuronal ensembles reconfigure for different tasks (multiple demand system). Not hardwired to one function.

**Connector hubs:** Dorsolateral PFC bridges between specialized networks, reconfiguring as demands shift.

**AI pattern:** Orchestrator sets goals and biases/routes, doesn't micromanage. Specialist agents handle domain implementation. Agents reconfigure for different tasks rather than fixed roles. Hub agents bridge between specialized networks.

### Neural Plasticity — Learning What Works
**Spike-Timing-Dependent Plasticity (STDP):** Presynaptic fires before postsynaptic (within 40ms) → synapse strengthens (LTP). Reverse order → synapse weakens (LTD). A causal learning rule — connections that predict outcomes are reinforced.

**Synaptic pruning:** During consolidation (sleep), weaker contacts pruned, potentiated contacts accumulated. Use-it-or-lose-it + selective replay.

**Homeostatic plasticity:** If a neuron becomes too active, all synaptic inputs globally scaled down. Too quiet → scaled up. Maintains stability while preserving learned relative differences.

**Cortical remapping:** After injury, neighboring regions take over functions. Cross-modal reassignment possible.

**AI pattern:** Strengthen connections that predict successful outcomes. Timing matters — causal predictions prioritized over stale info. Prune underperforming connections during consolidation. Homeostatic scaling prevents any agent from dominating. Graceful degradation — remaining agents compensate for lost ones.

### Population Coding — Distributed Consensus
No single neuron encodes a decision. Information distributed across populations for robustness. Brain decomposes decisions into parallel variables: vmPFC (preferences), pSTS/TPJ (group behavior), IPS (environmental structure), dACC (integration). Integration is Bayesian, not winner-take-all.

**Two-phase architecture:** Posterior cortex: graded evidence accumulation (deliberation). Frontal cortex: ballistic winner-take-all once threshold crossed (commitment).

**Theta-gamma multiplexing:** Multiple items represented in different gamma subcycles nested within theta. Creates ordered sequence code for coordinating distant regions.

**AI pattern:** Multiple agents encode different decision variables. Coordinator integrates via weighted combination. Graded deliberation → threshold-triggered commitment. Time-multiplexed coordination cycles for ordered processing.

### Excitatory-Inhibitory Balance — Preventing Cascading Failures
Glutamate (excitatory) + GABA (inhibitory) in constant balance. Inhibitory neurons are **specific** in what they suppress — targeting different neuronal components for precision suppression, not wholesale shutdown.

**Glutamate-GABA overflow valve:** Excess excitatory signal automatically recruits proportional inhibition.

**Strategic disinhibition:** Inhibitory neurons selectively decrease firing near reward locations, enhancing desired signals. Inhibition sharpens and highlights, not just dampens.

**AI pattern:** Every excitatory signal (task request, escalation) needs proportional inhibitory counterpart (rate limiting, backpressure). Targeted suppression, not global shutdown. Overflow mechanisms auto-increase throttling during spikes. Strategic disinhibition — temporarily removing constraints to accelerate progress in specific areas.

### Neuromodulators — Global State Signals
**Dopamine:** Rapid signed prediction error — fires more for better-than-expected, less for worse. Inverted-U dose response: too little impairs motivation, too much impairs precision.

**Serotonin:** Unsigned surprise response — rises for any unexpected event. Dissipates more slowly than dopamine. Can enhance or inhibit dopamine.

**Endocannabinoid meta-modulation:** Activity-dependent regulation of the regulators themselves.

**AI pattern:** Broadcast system-wide state signals. "Dopamine" = reward prediction errors adjusting all agents' learning. "Serotonin" = surprise/novelty triggering global attention shifts. Operate on slower timescale than individual agent messages. Critical: inverted-U principle — too much or too little global signal degrades performance.

---

## CROSS-SYSTEM SYNTHESIS — Universal Patterns for Agent Orchestration

### Pattern 1: STIGMERGIC COMMUNICATION
**All four systems** coordinate through environment/medium rather than direct point-to-point messaging. Ants use pheromones, immune cells use cytokines, bees use dances, neurons use neurotransmitters. The shared medium IS the coordination mechanism.
→ **Agent pattern:** Shared state (task board, event bus, files) as primary coordination mechanism. Agents read/write to shared environment, not to each other.

### Pattern 2: SIGNAL DECAY / AUTOMATIC EXPIRY
Pheromones evaporate. Dance vigor fades. Cytokines degrade. Synaptic potentials decay. In every system, signals have a built-in TTL (time to live).
→ **Agent pattern:** All signals, task priorities, and cached results should auto-expire. Nothing persists without active reinforcement.

### Pattern 3: QUORUM SENSING / THRESHOLD COMMITMENT
Ants, bees, T cells, and neurons all use thresholds — collective action triggers only when independently-arriving confirmations reach a critical density.
→ **Agent pattern:** Require N independent agent confirmations before committing to irreversible actions. Not majority vote — quorum sensing.

### Pattern 4: TWO-TIER RESPONSE (FAST + SLOW)
Innate + adaptive immunity. Reflex + deliberate neural processing. Fast scouts + slow consensus in bees. Every system has a fast generic response backed by a slow precise one.
→ **Agent pattern:** Rule-based first responders for known patterns + reasoning agents for novel situations. Fast system packages context for slow system.

### Pattern 5: POSITIVE FEEDBACK + NEGATIVE FEEDBACK + EXPLORATION
Every system balances amplification of good solutions (positive feedback), dampening of stale/bad ones (negative feedback/decay), and random exploration to avoid local optima. This trio appears in ant foraging, bee nest selection, immune clonal selection, and neural STDP.
→ **Agent pattern:** Reinforce successful approaches + auto-expire unsuccessful ones + maintain stochastic exploration. The three-part balance.

### Pattern 6: EMERGENT SPECIALIZATION WITHOUT CENTRAL ASSIGNMENT
Ant response thresholds, immune clonal selection, neural adaptive coding, and bee scout self-selection all produce division of labor without a dispatcher.
→ **Agent pattern:** Agents self-select tasks based on capability thresholds rather than being centrally assigned. Specialization emerges from differential responsiveness.

### Pattern 7: PROPORTIONAL SCALING
Immune clonal expansion, ant recruitment intensity, bee waggle dance vigor, and neural population firing rates all scale proportionally to stimulus intensity.
→ **Agent pattern:** Scale agent instances and priority proportionally to task urgency/size. Auto-scale up during demand, contract after resolution.

### Pattern 8: MULTI-TIER MEMORY
Immune memory cells (central, effector, tissue-resident, stem cell). Neural memory (working, short-term, long-term). Ant trail networks. Bee dance traditions.
→ **Agent pattern:** Memory at multiple tiers: central knowledge base, edge caches, agent-local context. Each tier trades off speed vs. completeness.

### Pattern 9: SAFETY THROUGH REDUNDANCY + SUPPRESSION
Immune two-signal requirement + Tregs. Neural E/I balance. Bee stop signals. Ant pheromone evaporation preventing lock-in.
→ **Agent pattern:** Dual-confirmation for destructive actions. Dedicated suppressor/monitor agents. Circuit breakers for runaway cascades.

### Pattern 10: GRACEFUL DEGRADATION
All four systems maintain function despite losing significant numbers of agents. Performance degrades linearly, not catastrophically.
→ **Agent pattern:** No single point of failure. System should degrade gracefully when agents are lost — remaining agents compensate through threshold adjustments.

---

**Sources cited:** ~60 academic papers, books, and articles across all four domains. Key references include Dorigo (ACO), Bonabeau (response threshold model), Deborah Gordon (harvester ants), Thomas Seeley (Honeybee Democracy, Princeton 2010), Seeley et al. (Science 2011 — bee/brain analogy), NCBI immunology textbooks, Nature Reviews Neuroscience, PNAS, and Frontiers in Neuroscience.

================================================================
AGENT: BACKEND
================================================================
## OUTBOX

### RESEARCH COMPLETE: Engineering Coordination Patterns for AI Agent Orchestration

---

## 1. Open Source Coordination at Scale

### Linux Kernel
- **Maintainer hierarchy**: Tree structure — Linus at top, ~100 subsystem maintainers, sub-maintainers, individual devs. Of 9,500 patches in a release, only 112 (1.3%) chosen directly by Linus. Everything flows up through trusted maintainers.
- **MAINTAINERS file**: Maps every file/directory to a named maintainer using glob patterns. Includes status codes (Supported, Maintained, Orphan). This is an active responsibility roster, not a credits list.
- **Merge window discipline**: Strict 2-week merge window after each release. Only code already reviewed and staged in subsystem trees is eligible. Miss it and you wait for the next cycle (6-10 weeks). No exceptions.
- **Patch provenance tags**: Every patch carries `Signed-off-by` (author certifies right to submit), `Reviewed-by` (technical correctness), `Acked-by` (maintainer acknowledges), `Tested-by`. Tags must be removed if patch changes substantially.
- **linux-next integration tree**: Stephen Rothwell aggregates all subsystem trees daily for pre-merge conflict detection. All patches should appear in linux-next before the merge window opens.

### Kubernetes
- **SIGs (Special Interest Groups)**: Every part of the codebase is owned by a SIG. Each has 1-2 chairs, a formal charter, and designated subproject owners.
- **OWNERS files**: Every directory contains a YAML file listing `approvers` and `reviewers`. The Prow bot auto-selects the minimum set of approvers needed for a PR. A PR cannot merge until every modified file is covered by an `/approve`.
- **Two-phase review**: Reviewer types `/lgtm` (code review) + Approver types `/approve` (ownership). Both labels required. Tide bot batches approved PRs and merges automatically.
- **Membership ladder**: Member (1+ merged PRs) → Reviewer (3+ months, 20+ PRs reviewed) → Approver (3+ months, 30+ PRs) → Subproject Owner. Zero contributions over 12 months = removal.
- **KEPs (Enhancement Proposals)**: Any user-facing change needs a structured proposal with motivation, design, risks, test plan. Iterated through community discussion.
- **Lazy consensus**: Proposals announced with a waiting period. No objections = approved. Prevents blocking on absent stakeholders.

### Apache Software Foundation
- **Role hierarchy**: User → Developer → Committer → PMC Member → PMC Chair → ASF Member → Board. Merit in one project does NOT transfer.
- **Voting**: Code changes need 3 binding +1 votes and ZERO -1 votes. A single -1 is an absolute veto (with technical justification required). Releases need majority +1 and cannot be vetoed. All votes run 72 hours minimum.
- **"Community over code"**: A healthy community can fix bad code; a toxic community kills good code.

### Cross-Cutting Patterns for Agent Protocol

| Pattern | How It Works | Agent Protocol Application |
|---------|-------------|---------------------------|
| **File-level ownership** | MAINTAINERS/OWNERS files map files to people | Every file/module has a single owning agent |
| **Multi-phase review** | Separate author, reviewer, approver roles | No agent both authors AND approves changes |
| **Integration trees** | linux-next tests combined work before merge | Test ALL concurrent agent work together before merging |
| **Time-boxed merge windows** | Miss the window, wait for next cycle | Define integration points, don't merge continuously |
| **Progressive trust** | Earn permissions through demonstrated reliability | New agents start limited; reliable agents get broader authority |
| **Veto power** | Any reviewer can block with justified objection | Any agent can block changes in its domain with stated reason |
| **Inactivity removal** | 12 months idle = removed from OWNERS | Stale agent files should be flagged |

---

## 2. Code Review at Scale

### Google's Process (Critique)
- **Three approval "bits"**: Every change needs (1) LGTM from a peer, (2) code ownership approval, (3) readability approval (language-specific certification). A single reviewer can satisfy all three.
- **75%+ of reviews involve exactly one reviewer**. Median CL: ~24 lines. Guideline: ~200 lines max. Median review latency: under 4 hours. 70% commit within 24 hours.
- **80% of reviews require the developer to take action** — not rubber stamps.
- **Philosophy**: "Approve if the change improves the overall codebase, even if imperfect." Continuous improvement over perfection.

### Microsoft's Process (CodeFlow)
- Team-configurable. Median of up to 4 reviewers (much more than Google).
- Sign-off policies vary by team. Some require all reviewers, others allow PM-only approval.
- Review latency: 14.7-24 hours (slower than Google's ~4h).

### What Code Review Catches vs Misses
- **Catches well**: Design inspections have 55-60% defect detection rate. 75% of findings improve evolvability (understandability/modifiability), not visible bugs.
- **Misses**: Only ~15% of comments identify actual defects. The other 85% are style/formatting/naming. Consistently misses: concurrency issues, integration failures, architectural mismatches, security vulnerabilities, performance regressions.
- **Opportunity for AI agents**: Automate the 85% (style/linting) and have specialized agents focus on the hard-to-catch 15%.

### CODEOWNERS (GitHub)
- Gitignore-style patterns mapping file paths to owners. Last match wins.
- Integrates with branch protection: "Require review from Code Owners."
- Approval from ANY listed owner satisfies the requirement.

### Patterns for Agent Protocol

| Pattern | Application |
|---------|-------------|
| **Google's "Three Bits"** | Specialized agent roles: correctness checker, ownership checker, style checker |
| **Small changes + fast feedback** | Break agent work into small reviewable units (~200 lines) |
| **"Attention Set"** | Clear handoff: which agent needs to act next |
| **Tiered review** | Tier 1 (fast/cheap agent) for basics, Tier 2 (capable agent) for architecture/security |
| **CODEOWNERS routing** | File-path patterns route changes to domain-specialist agents |

---

## 3. CI/CD as Independent Agent Patterns

### Checks as Independent Validators
Each CI check is structurally identical to an agent: narrow domain, isolated execution, clear pass/fail output.

| Check | Analogous Agent Role |
|-------|---------------------|
| Linter | Style cop — stateless, fast, opinionated |
| Type Checker | Contract enforcer — catches interface mismatches |
| Unit Tests | Domain specialist — tests individual components |
| Integration Tests | Systems thinker — tests boundaries |
| SAST (Semgrep/CodeQL) | Security reviewer — read-only analysis |
| SCA (Dependabot/Snyk) | Supply chain auditor |

**Key properties**: Independence (isolated containers), statelessness (clean environment each run), binary output (pass/fail), parallel execution. This IS the agent model.

### Fan-out / Fan-in (GitHub Actions)
- Independent jobs run in parallel by default. The `needs` keyword creates dependencies.
- Fan-in: deploy job waits for all checks to pass. If any fails, deploy is skipped.
- Matrix strategy multiplies parallelism (e.g., 3 OS × 3 Node versions = 9 parallel jobs).

### Merge Queues (Bors/Mergify/GitHub)
- Solve **merge skew**: two PRs pass CI independently but break when combined.
- **"Not Rocket Science Rule"** (Graydon Hoare): Maintain an automatically-tested branch that always passes all tests.
- Process: PR added to queue → temporary branch created with (main + all queued PRs + this PR) → CI runs → pass = merge, fail = eject.
- Advanced: **batching** (test multiple PRs together, bisect on failure), **speculative execution** (start testing PR #3 before PR #1 finishes), **priority queues** (hotfixes jump the line).

### Status Checks as Consensus Gates
- All required checks must pass (unanimous consent model).
- Merge button physically disabled until green.
- If branch falls behind main, checks must re-run.

### Trunk-Based Development + Feature Flags
- **TBD wins for agents**: Small frequent commits, immediate conflict detection, fast CI feedback, fine rollback granularity.
- **Feature flags** decouple deploy from release. Agents deploy behind flags — no blocking dependencies. Founder controls release order.

### Patterns for Agent Protocol

| Pattern | Application |
|---------|-------------|
| **Fan-out/Fan-in DAG** | Assign independent tasks simultaneously; integration waits for all |
| **Merge queue serialization** | Test each agent's output against combined state of all accepted work |
| **Unanimous consent** | All domain agents must approve before changes land |
| **Speculative execution** | Start Agent B assuming Agent A succeeds; re-run if A fails |
| **Feature flags** | Agents deploy behind flags; orchestrator controls release |
| **Confidence-threshold routing** | 0.90+ = auto-approve; 0.75-0.90 = human review; <0.75 = reject |

---

## 4. Incident Response / SRE Patterns

### Incident Command System (ICS) Roles

| Role | Responsibility | Agent Analogy |
|------|---------------|---------------|
| **Incident Commander** | Coordinates but NEVER directly fixes | Orchestrator agent — routes work, doesn't execute |
| **Deputy** | Hot standby, flags overlooked issues | Failover/watchdog agent |
| **Scribe** | Documents timeline, decisions, actions | Logging/audit agent |
| **Operations Lead** | ONLY group modifying the system | Executor agent(s) with write access |
| **Communications Lead** | Periodic updates to stakeholders | Status/notification agent |
| **Subject Matter Expert** | Domain diagnosis and fixes | Specialized domain agents |

**Key rule**: "The operations team should be the only group modifying the system during an incident." Prevents freelancing.

### IC Communication Protocol (PagerDuty)
- **Named assignment + time-box + acknowledgment**: "Bob, investigate X. I'll check back in 3 minutes. Understood?" NEVER "Can someone..." (bystander effect).
- **Veto-based consensus**: "Are there any strong objections?" (not seeking agreement, seeking blockers).
- **Severity disputes**: "We do not discuss incident severity during the call."
- **Executive override**: "Do you wish to take command?" (forces formal responsibility or silence).

### Severity → Autonomy Mapping

| Severity | Response | Agent Autonomy Level |
|----------|----------|---------------------|
| SEV-1 | Full incident protocol, page IC | Agent pauses, escalates to human |
| SEV-2 | Formal incident, page IC | Agent pauses, escalates to human |
| SEV-3 | Page service team, monitor | Agent acts but notifies human |
| SEV-4 | First priority above routine | Agent handles autonomously |
| SEV-5 | Create ticket | Agent handles autonomously |

**Critical rule**: "If unsure which level, treat it as the higher one."

### Runbooks as Agent Task Files
Three-layer architecture:
1. **Triage**: Alert arrives → classify → route to right handler
2. **Diagnostics**: Auto-fetch context (deployment history, metrics, related files)
3. **Remediation**: Present options with risk assessment; approval gates for destructive actions

Agent `.md` files ARE runbooks. The three layers map to: understand context → gather information → act with appropriate approval gates.

### Blameless Postmortems
- Trigger: user-visible downtime, data loss, on-call intervention, monitoring failure
- **Five Whys**: Keep asking until reaching systemic causes. "The root isn't a person's mistake — it's missing quality controls, unclear ownership, or mismatched incentives."
- Template: Summary → Timeline → Root Cause + Contributing Factors → Impact → Lessons → Action Items (each with owner + deadline)
- After agent task failures: auto-generate structured postmortem focusing on system gaps, not "agent X failed"

### ChatOps = Agent Protocol's .md Files
- Single channel for all communication, time-stamped and logged
- Bot auto-creates incident channels, posts context, pages responders
- New responders read channel history to get up to speed
- The OUTBOX/INBOX pattern in the Agent Protocol IS ChatOps

---

## 5. Pair Programming & Mob Programming

### Driver/Navigator Pattern
- **Driver**: Controls keyboard, thinks tactically, narrates actions
- **Navigator**: Reviews in real-time, thinks strategically, parks future notes
- Switch roles every 25 minutes. Max sustainable: ~6 hours/day.
- **Agent mapping**: One agent writes code (driver) while another reviews in real-time and maintains strategic context (navigator).

### Strong-Style Pairing
- **Rule**: "For an idea to go from your head into the computer, it MUST go through someone else's hands."
- Navigator makes all decisions, driver executes. When driver has an idea, they must hand off and become navigator.
- **Three abstraction levels**: Intent ("validate user input") → Location ("open validation module") → Details ("type `if (!input.trim()) return null`")
- **Agent mapping**: Orchestrator agent decides what to build (navigator). Executor agent implements but cannot independently deviate (driver). Forces all decisions through explicit communication.

### Mob Programming
- **Three roles**: Driver (typist, doesn't code independently), Navigator (decides what to create), Mob (generates ideas, reviews, researches)
- Rotation: 3-4 people = 30 min; 5+ = 15 min; remote = 10 min
- **Agent mapping**: Multiple specialist agents on one task. Coding agent drives, planning agent navigates, researcher agent looks up docs, QA agent reviews for bugs, product agent checks requirements.

### Ping-Pong TDD Pairing
- Agent A writes a failing test → Agent B writes minimum code to pass → Agent B writes next test → Agent A implements → Repeat
- **This is the strongest pattern for multi-agent coordination**: The test IS the contract. Prevents scope creep. Creates automatic verification. Clear handoff protocol.
- **Proven with AI**: Mechanical Orchard ran two GPT-4 personas in ping-pong TDD to build a banking system. Worked for small tasks at ~$6/iteration.

### Async Baton-Pass
- Synchronous pairing session → one dev continues solo with notes → other picks up next day
- **Agent mapping**: Agent A works, writes structured handoff to OUTBOX, Agent B picks up using that context. The agent `.md` file IS the baton.

### Effectiveness Research

| Finding | Source |
|---------|--------|
| Pairs take 15% more effort but produce 15% fewer bugs | Williams (2000) |
| Pairs complete tasks 40% faster with higher confidence | Nosek (1998) |
| 175 lines/person-month for pairs vs 77 for solo; errors 3 orders of magnitude lower | Jensen (1996) |
| Complex tasks: significantly higher quality at greater effort | Hannay meta-analysis |
| Simple tasks: pairing overkill; complex tasks: pairing essential | Consensus |

**Key insight**: The 15% coordination overhead is worth it for complex tasks. For AI agents, a review agent watching a coding agent catches hallucination-driven bugs. Simple boilerplate doesn't need multi-agent coordination; complex architectural work benefits significantly.

---

## SYNTHESIS: Top Patterns for Agent Protocol v4

### The 7 Most Actionable Patterns

1. **OWNERS-style routing**: Machine-readable files declaring which agent owns each module, who can review, and escalation paths. (From Linux MAINTAINERS + K8s OWNERS + GitHub CODEOWNERS)

2. **Signed-off-by provenance chain**: Every change carries `Created-by: agent-X`, `Reviewed-by: agent-Y`, `Approved-by: cowork`. No change merges without this chain. (From Linux kernel)

3. **Merge queue for agent output**: Test each agent's output against the combined state of all previously accepted work before committing. Prevents merge skew. (From Bors/Mergify)

4. **Ping-pong TDD between agents**: One agent writes tests (the spec), another implements. The test suite is the shared contract. Most directly applicable pattern for AI agents. (From pair programming research + Mechanical Orchard AI experiment)

5. **Severity → autonomy mapping**: SEV-4/5 = agent acts autonomously. SEV-3 = act but notify. SEV-1/2 = pause and escalate. "When in doubt, escalate up." (From PagerDuty/Google SRE)

6. **Strong-style orchestrator/executor**: Orchestrator decides at three abstraction levels (intent/location/details). Executor implements but cannot independently deviate. Creates audit trail. (From strong-style pairing)

7. **Fan-out/fan-in DAG**: Independent agent tasks run in parallel. Integration task waits for all to complete. The `needs` graph defines execution order. (From GitHub Actions)

================================================================
AGENT: DESIGNER
================================================================
## OUTBOX

### RESEARCH COMPLETE: Human Organization Patterns for AI Agent Orchestration

I researched five high-performance human coordination systems: ER teams, film production, newsrooms, kitchen brigades, and orchestras. Below are the findings organized by domain, followed by a cross-domain synthesis of the most powerful patterns.

---

## 1. HOSPITAL EMERGENCY ROOMS

### Hospital Incident Command System (HICS)
- **Single Incident Commander** with absolute authority; can override normal policies during emergencies
- **4 Section Chiefs** own domains: Operations (patient care), Planning (action plans), Logistics (supplies), Finance (costs)
- **Span of control: 3-7 direct reports per leader** (ideal = 5). This is a hard rule.
- **Unity of command:** every person reports to exactly ONE supervisor
- **Scalable activation:** roles scale up/down — a small hospital may have one person filling multiple roles; unused roles stay dormant
- **Planning in time horizons:** 4-hour, 8-hour, 24-hour, and 48-hour action plans

### Triage Systems
- **START Triage** (mass casualty): assess in <60 seconds using RPM (Respiration, Perfusion, Mental status). Four categories: BLACK (dead), RED (immediate), YELLOW (delayed), GREEN (walking wounded)
- **ESI (Emergency Severity Index):** 5 levels based on resource needs — ESI-1 (life-saving intervention NOW) down to ESI-5 (zero resources needed)
- **Manchester Triage:** 5 levels with HARD TIME LIMITS — Immediate: 0 min, Very Urgent: 10 min, Urgent: 60 min, Standard: 120 min, Non-urgent: 240 min

### Communication Protocols
- **Closed-Loop Communication (3 steps):** (1) Sender transmits to a NAMED person, (2) Receiver repeats back the FULL message, (3) Sender confirms or corrects. Every. Single. Time.
- **Call-Out:** Leader broadcasts critical info AND directs a specific action to a named individual: "Patient is in V-fib. Sarah, prepare the defibrillator."
- **CUS Escalation Words:** "I'm Concerned" → "I'm Uncomfortable" → "This is a Safety issue" (each word escalates urgency)
- **Two-Challenge Rule:** If a concern is ignored, voice it at least twice; if still ignored, escalate up the chain

### Handoff Protocols
- **SBAR:** Situation → Background → Assessment → Recommendation. Used for quick escalations.
- **I-PASS:** Illness severity → Patient summary → Action list → Situation awareness → Synthesis. KEY DIFFERENCE: I-PASS requires the RECEIVER to synthesize and read back. Reduces errors more than SBAR.

### Role Clarity (Crisis Resource Management)
- **Team Leader is HANDS-OFF** — does NOT perform procedures. Maintains situational awareness and orchestrates.
- Roles assigned by name: "John, you are airway. Maria, you are compressions."
- Assignee VERBALLY acknowledges their role
- Leader periodically "thinks out loud" to maintain the shared mental model

### Surge Protocols (3 tiers)
- **Tier 1 (Conventional):** Max existing resources, cancel electives, accelerate discharges. Target: +20% capacity.
- **Tier 2 (Contingency):** Repurpose non-standard spaces, adapt staffing ratios. Care is "functionally equivalent" but non-standard. Target: +100% capacity.
- **Tier 3 (Crisis):** Non-clinical spaces used, staff work outside normal scope. Triage shifts to allocation-based. Target: +200% capacity.
- **4S model:** Staff, Stuff (supplies), Space, System — all four must scale together.

---

## 2. FILM PRODUCTION

### Department Structure
- 15+ departments, each a self-contained unit with internal hierarchy (Head → Best Boy → Crew)
- Departments are like microservices: own their domain completely, expose a single interface (department head) to production
- **Above-the-line** (creative/financial leadership) vs **below-the-line** (execution)

### Chain of Command
- **Director** (creative vision) → **1st AD** (operational coordination) → **Department Heads** → Crew
- Information flows DOWN through department heads. A grip never talks directly to the director.
- **The Director Trifecta:** Director + 1st AD + DP meet daily to reconcile creative goals with logistics

### The 1st AD — The Real Orchestrator
- Builds master shooting schedule (the stripboard)
- Creates daily plan → 2nd AD generates call sheets
- Calls the roll (verbal protocol initiating each take)
- Manages set tempo — decides when to push, when to cut losses
- Guards the director's creative space by absorbing ALL logistics
- Named verbal commands: "Final checks" → "Picture is up!" → "Quiet on set!" → "Roll sound!" → "Roll camera!" → "Marker!" → "Action!" → "Cut!" → "Moving on!"

### Call Sheets — Single Source of Truth
- One document coordinates 100+ people daily. Distributed night before (async).
- Contains: scene list, cast call times, department-specific notes, locations, weather, advance schedule
- **Color-coded revision system:** white → blue → pink → yellow → green → goldenrod. Everyone instantly sees if they have the current version.

### Sequential Readiness Gates (Roll Protocol)
Each take follows a strict sequence. Each step is a BLOCKING GATE — cannot proceed without confirmation:
1. "Final checks" → departments adjust
2. "Last looks!" → hair/makeup/wardrobe
3. "Picture is up!" → all activity stops
4. "Quiet on set!" → silence
5. "Roll sound!" → "Sound speed!" (confirmation)
6. "Roll camera!" → "Speed!" (confirmation)
7. "Marker!" → slate clap
8. "Set!" → camera confirms
9. "Action!" (Director only)
10. "Cut!" (Director only)

### Pre-Production Eliminates Chaos
- **Script Breakdown:** every element tagged by category (cast, props, SFX, etc.)
- **Stripboard:** master schedule, scenes rearranged to optimize logistics
- **Same breakdown, department-specific extraction:** each department reads the same spec and extracts what's relevant to their domain

### Department Autonomy
- **Dedicated radio channels:** Ch1 Production, Ch3 Transport, Ch6 Camera, Ch7 Electric, etc.
- Department heads as single interface prevents N-to-N communication explosion
- **"Moving on" protocol:** every department independently executes its own transition plan in PARALLEL, converging on the same deadline

---

## 3. NEWSROOMS

### Editorial Hierarchy
- Editor-in-Chief → Managing Editor → Section Editors → Reporters
- **Hub-and-spoke with domain specialization.** Section editors act as middleware — buffer, filter, and translate between strategic layer (EIC) and execution layer (reporters).
- Information flows UP (pitches, field intel), gets FILTERED at section editor level, PRIORITIZED at planning meeting, then flows back DOWN as assignments

### Story Budget — The Central Coordination Artifact
- Structured list of every planned story with fields: **Slug** (1-4 word ID), budget line, reporter, expected length, art needs, deadline, status
- **Budget meeting:** each section editor presents their section's budget. Group calibrates — stories promoted, demoted, held, or killed. Resource conflicts resolved.
- This is a shared task queue with structured metadata + synchronization point

### Copy Desk — Pipeline with Quality Gates
- **Stage 1:** Reporter files draft to section editor
- **Stage 2:** Substantive edit (structure, narrative, completeness)
- **Stage 3:** Copy desk — **Slot/Rim pattern:** the Slot (desk chief) dispatches stories to Rim editors (line-level editing), then does a final slotting review. This is a **fan-out/fan-in pattern.**
- **Stage 4:** Page layout/production
- **Stage 5:** Final senior editor sign-off

### Breaking News = Priority Interrupt
- Same structure, different tempo — the workflow compresses, not transforms
- Single authority (news desk / senior editor) triggers the interrupt and reallocates resources
- Stories can be KILLED at any pipeline stage. Sunk cost is accepted.

### Fact-Checking Models
- **Newspaper model:** reporters verify their own facts (fast, lower quality)
- **Magazine model (The New Yorker):** dedicated fact-checkers. Writer ANNOTATES every factual claim with its source. Checker independently verifies each claim. Writer and checker work in PARALLEL via the annotation system.
- **Hybrid model:** use newspaper model for time-sensitive pieces, magazine model for investigations. Variable QA depth based on task criticality.

### Deadline-Driven Coordination
- **Cascading deadlines:** each pipeline stage has its own deadline feeding the next
- **"Drop dead" deadline:** absolute final moment. After this, nothing goes in. Period.
- **Late stories compress downstream work** — the system explicitly acknowledges this quality tradeoff
- Stories that won't make deadline are HELD for next edition rather than rushed

### The Slug System
- Human-readable, 1-4 word task ID (e.g., "HURRICANE IAN", "MAYOR RACE")
- Prefix codes embed metadata: "AM" = morning edition, "CX" = correction
- Used by everyone from reporter to press operator — universal, memorable, speakable

---

## 4. KITCHEN BRIGADE SYSTEM

### Brigade de Cuisine Hierarchy (Escoffier)
- **Chef de Cuisine** → **Sous Chef** → **Chefs de Partie** (station cooks) → **Commis** (apprentices)
- Stations: Saucier (sauces), Poissonnier (fish), Rotisseur (roast), Grillardin (grill), Friturier (fry), Entremetier (vegetables), Garde Manger (cold), Patissier (pastry), Boucher (butcher)
- **Tournant (Swing Cook):** no fixed station — fills in anywhere. The deliberate generalist in a specialist system.
- **Design principle:** every task has exactly ONE owner. No overlap, no ambiguity.

### The Pass System
- All completed plates converge at a single physical counter (the pass)
- The **Aboyeur/Expeditor** stands at the pass — performs final quality check on EVERY plate
- Only after expeditor approves does food leave the kitchen
- "Dying on the pass" = completed food going cold because other items from the same table are late (staleness detection)

### Call-and-Response Protocol
- **"Oui, Chef!"** — mandatory response to any directive. Confirms heard AND understood.
- **"Heard!"** — general acknowledgment for any information
- **Echo/read-back:** stations must repeat orders back. Expeditor repeats by NAME until acknowledged.
- **Safety calls:** "Behind!" "Corner!" "Hot!" "Sharp!" — mandatory, never skipped.
- **Key rule:** silence is NEVER acceptable. Every communication is a closed loop.

### Station Ownership
- Each Chef de Partie has COMPLETE ownership: full accountability, full authority, full knowledge
- When expeditor calls "Fire 2 salmon" — zero ambiguity about who acts
- Two people never own the same output

### Mise en Place ("Everything in Its Place")
- All ingredients washed, cut, measured, positioned BEFORE service begins
- All tools in consistent, memorized locations
- Transforms service from problem-solving into pure execution
- "Your hands know exactly where to reach. You're cooking, not treasure hunting."
- **Principle:** front-load all preparation so execution-time complexity is minimized

### Ticket/Order System
- Orders flow: Server → POS → Ticket printer → Expeditor → Kitchen
- **"All Day" counts:** running totals across all active tickets ("Five meatloaf all day" = five total across all orders)
- **15-Minute Rule:** if 15 min pass without firing next course, check status
- Expeditor may HOLD tickets if stations are overloaded ("dragging the table")

### "Fire" Timing — The Core Orchestration Mechanism
- **Two-phase call:** "Ordering" (awareness — stations note but DON'T cook) → "Fire" (action — begin cooking NOW)
- Expeditor knows cook times for every dish and staggers fire calls so all dishes for a table hit the pass simultaneously
- **Priority overrides:** "On the fly!" (highest — remake NOW), "Rail it!" (rush)

### Handling "The Weeds" (Overload)
1. **Acknowledge** — cook must call out they're behind. Silence is worst response.
2. **Stop the bleeding** — finish what's cooking before starting new items
3. **Expeditor throttles input** — holds new tickets, stops calling to overwhelmed stations
4. **Redistribute** — Sous Chef or Tournant jumps onto struggling station
5. **Slow the whole system** — host stops seating, servers delay. Restaurant throttles throughput.
6. **Key insight:** response to overload is to REDUCE INPUT, not demand faster output. Backpressure.

---

## 5. ORCHESTRAS

### The Conductor's Role
- 90% preparation, 10% performance-night gestures
- **Score study (weeks before):** analyzes structure, makes binding decisions on tempo, dynamics, phrasing
- **Rehearsal leadership:** runs rehearsals with detailed plan, diagnoses ensemble problems, communicates corrections
- **Performance:** single synchronization point — signals starts/stops, maintains rhythm, cues entrances, shapes expression
- **Key pattern:** the conductor holds the ONLY full score. Every musician only sees their own part. The conductor is the sole agent with complete system state.

### Section Leaders as Intermediaries
- **Concertmaster** (principal 1st violin): second-in-command. Translates conductor's artistic intent into technical language for strings. Sets bowings. Leads tuning. Provides continuity across guest conductors.
- **Principal players** (one per section): play solos, set style/tone, lead by example
- **Communication flow:** Conductor → Concertmaster → String principals → Section members. Conductor → Wind/brass principals → Section members.
- **Two-tier delegation:** conductor never micromanages individual players

### The Score as Shared State (Asymmetric Information Architecture)
- **Full score** (conductor only): shows every instrument simultaneously. Complete system state.
- **Individual parts** (one per musician): ONLY their line. No visibility into what others play.
- Musicians coordinate through: rehearsals (learning how their part fits), listening, following conductor, conventions ("I enter after the oboe solo")

### Rehearsal Protocols (Progressive Integration)
- **Individual preparation** (pre-rehearsal): parts distributed 2+ weeks early. Musicians expected to arrive PREPARED.
- **Sectional rehearsals:** strings together, winds together. Solve intra-section coordination.
- **Full orchestra rehearsals:** typically 2-4 sessions for professionals
- **Dress rehearsal:** complete run-through under performance conditions
- **9:1 playing-to-discussion ratio** — minimize coordination overhead, maximize execution

### Gestural Communication (Two-Channel Protocol)
- **Right hand** (baton): maintains beat pattern. The ictus (bottom point of each beat) is the universal sync signal.
- **Left hand:** expressive communication — crescendo, diminuendo, character
- **Cueing protocol (2 phases):** (1) Make eye contact with section about to enter, (2) Execute preparatory gesture. Engage, then signal.

### Self-Coordination Within Sections
- **Unified bowings:** pre-agreed rules set by concertmaster, marked into parts before rehearsal
- **Peripheral vision:** players watch principal's bow movements while reading music
- **Acoustic feedback:** players match volume, tone, timing to principal and stand partner
- Conductor does NOT manage intra-section coordination

### Chamber Music (Conductorless Coordination)
- Works for groups of ~3-8 players
- Coordination via: synchronized breathing (natural "start" signal), continuous eye contact, deep trust
- **The Orpheus Process** (named system): (1) Put power in the hands of workers, (2) Encourage individual responsibility, (3) Clarity of roles, (4) Share/rotate leadership, (5) Horizontal teamwork, (6) Learn to listen/talk, (7) Seek consensus, (8) Passionate dedication
- **Threshold rule:** below ~25 players, conductorless works. Above that, a dedicated conductor becomes necessary.

---

## CROSS-DOMAIN SYNTHESIS: Universal Patterns for AI Agent Orchestration

After analyzing all five domains, these patterns appear across THREE OR MORE systems:

### Pattern 1: HANDS-OFF ORCHESTRATOR
- **ER:** Team leader is hands-off — does NOT perform procedures
- **Film:** 1st AD coordinates but doesn't operate cameras or lights
- **Kitchen:** Expeditor calls orders and checks quality but doesn't cook
- **Orchestra:** Conductor doesn't play an instrument
- **Rule:** The coordinator must NOT do the work. Their job is maintaining system state, sequencing, and quality gating. The moment they start doing work, they lose situational awareness.

### Pattern 2: SINGLE INTERFACE PER DOMAIN (Department Head Pattern)
- **ER:** Section Chiefs own domains; IC talks to chiefs, not nurses
- **Film:** Department Heads are the sole interface; crew never talks to director
- **Newsroom:** Section Editors buffer between EIC and reporters
- **Kitchen:** Chefs de Partie own stations completely
- **Orchestra:** Principal players lead their sections
- **Rule:** Every autonomous unit exposes ONE point of contact. Internal complexity is hidden. This prevents N-to-N communication explosion.

### Pattern 3: CLOSED-LOOP COMMUNICATION
- **ER:** Send → Read-back → Confirm (3-step protocol)
- **Kitchen:** Call → "Heard!" / Echo → Expeditor confirms
- **Film:** "Roll sound!" → "Sound speed!" / "Roll camera!" → "Speed!"
- **Rule:** No message is considered received until explicitly acknowledged. Silence is never acceptable. This eliminates the most dangerous failure mode: assumed understanding.

### Pattern 4: SINGLE SOURCE OF TRUTH DOCUMENT
- **Film:** Call sheet coordinates 100+ people from one document
- **Newsroom:** Story budget is the central coordination artifact
- **Orchestra:** The full score is the complete system state
- **Kitchen:** The ticket rail is the ordered queue of all active work
- **Rule:** One canonical document/artifact that everyone references. Different roles extract different information from it.

### Pattern 5: SEQUENTIAL READINESS GATES
- **Film:** The roll protocol — 10 steps, each a blocking gate requiring confirmation
- **ER:** HICS activation checklist; triage before treatment
- **Kitchen:** "Ordering" (awareness) → "Fire" (action) — two-phase commit
- **Orchestra:** Cueing protocol — engage (eye contact) → signal (preparatory gesture)
- **Rule:** Before execution, each subsystem must explicitly confirm readiness in a defined order. No step proceeds without acknowledgment.

### Pattern 6: PREPARATION ELIMINATES CHAOS (Mise en Place Principle)
- **Kitchen:** Mise en place — all prep done before service
- **Film:** Pre-production (script breakdown, stripboard) eliminates on-set chaos
- **Orchestra:** Score study + individual preparation before rehearsals
- **Newsroom:** Story budgets planned in morning meetings before deadline pressure
- **Rule:** Front-load all preparation so execution-time complexity is minimized. During execution, agents should be combining pre-staged components, not doing raw preparation.

### Pattern 7: EXPLICIT ROLE ASSIGNMENT BY NAME
- **ER:** "John, you are airway. Maria, you are compressions." + verbal acknowledgment
- **Kitchen:** Expeditor calls station by name until acknowledged
- **Film:** 1st AD directs commands to specific department heads
- **Rule:** Never broadcast a task to "someone." Assign to a NAMED agent. The assignee must verbally/explicitly confirm acceptance.

### Pattern 8: BACKPRESSURE / THROTTLING
- **Kitchen:** Expeditor holds tickets when stations are overloaded. Host stops seating. Response to overload = REDUCE INPUT.
- **ER:** Surge protocols scale capacity in tiers. Ambulance diversion when full.
- **Newsroom:** Stories held for next edition rather than rushed. "Drop dead" deadline = hard cutoff.
- **Rule:** When overwhelmed, reduce input rate — don't demand faster output. The system must have explicit mechanisms to throttle incoming work.

### Pattern 9: PRIORITY INTERRUPT SYSTEM
- **Kitchen:** "On the fly!" overrides everything
- **Newsroom:** Breaking news triggers pipeline pivot — same structure, compressed tempo
- **ER:** Triage continuously re-prioritizes; incoming critical patient bumps queue
- **Rule:** The system must support preemption. A single authority can interrupt normal flow and reallocate resources. Work-in-progress can be killed without guilt.

### Pattern 10: PROGRESSIVE INTEGRATION
- **Orchestra:** Individual practice → Sectional rehearsal → Full rehearsal → Dress rehearsal
- **Film:** Script breakdown → Pre-production → Shoot day
- **ER:** Individual competency → Team drills → Full-scale exercises
- **Rule:** Build from parts to whole. Test at each integration level. Problems found early (in smaller scope) are cheaper to fix.

### Pattern 11: SPAN OF CONTROL LIMITS
- **ER (HICS):** 3-7 direct reports per leader (ideal = 5)
- **Orchestra:** Conductor threshold ~25 players; above that, section leaders become necessary
- **Film:** Department heads manage their crew; 1st AD manages department heads (not individual crew)
- **Rule:** No single coordinator should manage more than ~7 direct agents. Beyond that, introduce intermediate leaders.

### Pattern 12: THE TOURNANT / SWING ROLE
- **Kitchen:** Tournant has no fixed station — fills in wherever needed. The deliberate generalist.
- **ER:** Float nurses and on-call specialists
- **Film:** Production Assistants (PAs) as general-purpose runners
- **Newsroom:** General assignment reporters
- **Rule:** In a system of specialists, you need at least one explicit generalist who can cover any role during surges or failures.

### Pattern 13: QUALITY GATE AT OUTPUT
- **Kitchen:** The pass — every plate inspected by expeditor before leaving
- **Newsroom:** Copy desk + senior editor sign-off before publication
- **Film:** "Check the gate" after each take; dailies review
- **Rule:** No output reaches the customer/user without passing through a quality checkpoint. This is the last line of defense.

### Pattern 14: ASYMMETRIC INFORMATION (Need-to-Know)
- **Orchestra:** Musicians see only their own part; conductor sees the full score
- **Film:** Each department gets the same script breakdown but extracts only their relevant items
- **Kitchen:** Each station only hears the items relevant to them from the ticket
- **Rule:** Agents receive only task-relevant context. Only the orchestrator needs complete system state. This prevents information overload and keeps agents focused.

### Pattern 15: SURGE / DEGRADED MODE
- **ER:** 3-tier surge (conventional → contingency → crisis). Crisis mode = degraded but deliberate.
- **Kitchen:** "In the weeds" protocol — throttle, redistribute, slow the whole system
- **Newsroom:** Breaking news compresses the pipeline; quality tradeoffs are explicit
- **Rule:** The system must have named modes for operating under stress. Each mode has explicit tradeoffs (what quality is sacrificed for speed). The transition between modes is a deliberate decision, not a gradual slide.

---

### RECOMMENDED NEXT STEPS FOR AGENT PROTOCOL

Based on this research, these are the highest-impact patterns to incorporate:

1. **Adopt the Expeditor model** — the orchestrator (Cowork) should be explicitly hands-off. Never assign Cowork a building task.
2. **Add closed-loop communication** — agents must acknowledge tasks before starting. "Heard!" equivalent.
3. **Formalize the two-phase commit** — "Ordering" (agent receives awareness of upcoming work) → "Fire" (explicit trigger to begin). This prevents premature execution.
4. **Add backpressure rules** — if an agent reports being overwhelmed or blocked, the system should throttle new assignments, not push harder.
5. **Enforce span of control** — no more than 5-7 agents reporting to one coordinator. Beyond that, add intermediate leads.
6. **Add the Tournant role** — designate one agent as a generalist swing agent that can cover any domain.
7. **Adopt the slug system** — give every task a human-readable 1-4 word identifier that persists across its lifecycle.
8. **Add mise en place phase** — before execution begins, require agents to confirm their "prep" is complete (dependencies resolved, context loaded, contracts read).
9. **Define surge modes** — name the degraded operating modes so the team can deliberately shift between them.
10. **Add the quality gate** — no agent output ships to the user without passing through a review checkpoint.

