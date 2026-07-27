# AI Coding Workflow

Inspired by **Matt Pocock's** skills — real engineering, not vibe coding.

Uses **OpenCode Go** subscription ($5 first mo → $10/mo, $60/mo usage cap).
All skills from [`mattpocock/skills`](https://github.com/mattpocock/skills).

---

## Model Reference

Pricing via OpenCode Go. "Req/5h" = estimated requests per 5-hour rolling window.

| Model | Input $/1M | Output $/1M | Req/5h | Req/mo | Context | Key Strength |
|---|---|---|---|---|---|---|
| **Qwen3.7 Max** | $2.50 | $7.50 | 950 | 4,770 | 1M | Highest SWE-bench Pro in Go (60.6%). Best for hard planning. |
| **DeepSeek V4 Pro** | bundled | bundled | 3,450 | 17,150 | 1M | LiveCodeBench 93.5%, Codeforces 3206. Best for implementation. |
| **Kimi K2.6** | $0.95 | $4.00 | 1,150 | 5,750 | 256K | Agent Swarm (300 sub-agents). Best for agentic coding. |
| **Kimi K2.7 Code** | $0.95 | $4.00 | 1,350 | 6,750 | 256K | Coding-focused. More req than K2.6. Great mid-tier planner. |
| **DeepSeek V4 Flash** | $0.14 | $0.28 | **31,650** | 158,150 | 1M | Default workhorse. 79% SWE-bench Verified. Cheap. Fast. |
| **MiMo V2.5** | $0.14 | $0.28 | 30,100 | 150,400 | 1M | Budget workhorse. Same price as Flash, 1M context. |
| **MiniMax M2.5** | $0.30 | $1.20 | 6,300 | 31,500 | 205K | Best cost-per-benchmark-point (80.2% SWE-bench at $0.30). |

---

## Pipeline: Use Cases & Stage-Specific Routing

### 1. NEW PROJECT (from scratch)

The full pipeline — every stage matters when you're building something new.

```
[Triage] → [Interview] → [Domain Model] → [Spec] → [Tickets] → [Implement] → [Review]
```

| Stage | Command | Agent | **Model** | Why this model | Est. requests | Est. cost |
|---|---|---|---|---|---|---|
| **1. Triage** | `/triage` | Plan | **K2.7 Code** | Reading and categorizing issues. Needs good instruction-following, not max reasoning. 1,350 req/5h is plenty. | 1-2 | ~$0.06 |
| **2. Interview** | `/grill-with-docs` "interview me about [PROJECT]" | Plan | **K2.7 Code** | Research-heavy — reading docs, asking questions, exploring codebase. No code written. K2.7 Code's 256K context handles doc ingestion. Flash is cheaper but K2.7 Code gives better structured output for the domain model step. | 3-8 | ~$0.25 |
| **3. Domain Model** | `/domain-modeling` (called by grill-with-docs) | Plan | **same as above** | Builds CONTEXT.md glossary & ADRs. Tightly coupled to interview — same session, same model. | (included) | — |
| **4. Spec** | `/to-spec** | Plan | **Qwen3.7 Max** | Synthesis of everything discussed into a formal spec. This needs the best reasoning — Qwen3.7 Max has 60.6% SWE-bench Pro (highest in Go). The $2.50/$7.50 price is worth it for a correct spec. Only ~1 req. | 1-2 | ~$0.08 |
| **5. Tickets** | `/to-tickets` | Plan | **V4 Flash** | Breaking spec into tracer-bullet tickets is mechanical — find vertical slices, wire blocking edges. Flash at $0.14/$0.28 is more than sufficient. | 3-5 | ~$0.01 |
| **6. Implement** | `/implement` (+ `/tdd`) | **Build** (Tab to switch) | **DeepSeek V4 Pro** or **Kimi K2.6** | This is where real code gets written. V4 Pro: 1M context for large codebases, LiveCodeBench 93.5%. K2.6: Agent Swarm for complex multi-file changes. 3,450 req/5h on V4 Pro is generous. | 5-15 | ~$0.26 |
| **7. Code Review** | `/code-review` | Plan | **V4 Flash** | Reading diffs + checking against standards/spec. Flash at 79% SWE-bench Verified handles code review perfectly. Never waste an expensive model here. | 2-4 | ~$0.01 |

**Total per feature:** ~20-40 requests, **~$0.70** of your $60 monthly budget.

---

### 2. ADDING A FEATURE (to existing project)

Lighter — skips spec unless the feature is complex.

```
[Triage] → [Interview] → [Implement] → [Review]
```

| Stage | Command | Agent | **Model** | Why |
|---|---|---|---|---|
| **Triage** | `/triage` | Plan | **K2.7 Code** | Quick categorization. |
| **Interview** | `/grill-with-docs` "interview me about [FEATURE]" | Plan | **K2.7 Code** | Research the existing codebase + what needs to change. K2.7 Code is good at understanding code structure. |
| **Implement** | `/implement` | **Build** | **V4 Pro** or **K2.6** | Feature implementation. If the feature is straightforward (single file, simple logic), **V4 Flash** is fine — escalate only if Flash struggles. |
| **Review** | `/code-review` | Plan | **V4 Flash** | Cheap review pass. |

**Model escalation rule for implement:**
- Single function / small change → **V4 Flash** ($0.14/$0.28)
- Multi-file feature → **V4 Pro** (1M ctx, 93.5% LiveCodeBench)
- Complex agentic work (many coordinated changes) → **Kimi K2.6** (Agent Swarm)

---

### 3. BUG FIX

Matt Pocock's `/diagnosing-bugs` skill is a 6-phase discipline. Use it.

```
[Diagnose] → [Fix] → [Review]
```

| Stage | Command | Agent | **Model** | Why |
|---|---|---|---|---|
| **Phases 1-2: Build loop + Reproduce** | `/diagnosing-bugs` | **Build** | **V4 Flash** | Building test harnesses, running repro scripts. This is mechanical — writing code to catch the bug. Flash is fast and cheap. |
| **Phases 3-4: Hypothesise + Instrument** | `/diagnosing-bugs` | Plan | **K2.7 Code** or **K2.6** | Requires reasoning about what could cause the bug. K2.7 Code is fine for most bugs. Escalate to K2.6 or Qwen3.7 Max for hard heisenbugs. |
| **Phase 5: Fix** | (manual code change) | **Build** | **V4 Pro** | Writing the actual fix. Use Pro for correctness. |
| **Phase 6: Cleanup + regression test** | (manual) | **Build** | **V4 Flash** | Removing debug logs, committing. Cheap. |
| **Review** | `/code-review` | Plan | **V4 Flash** | Cheap review. |

**Key insight from Matt's skill:** Phase 1 (build a feedback loop) is the most important step. Don't move to hypothesizing until you have a tight red/green signal. Use **V4 Flash** here because you'll iterate many times — cost adds up fast.

---

### 4. ARCHITECTURE REDESIGN

Requires the heaviest models for planning.

```
[Architecture Scan] → [Spec] → [Tickets] → [Implement] → [Review]
```

| Stage | Command | Agent | **Model** | Why |
|---|---|---|---|---|
| **Architecture scan** | `/improve-codebase-architecture` | Plan | **Qwen3.7 Max** | This produces an HTML report of deepening opportunities. Needs the deepest reasoning — Qwen3.7 Max has 60.6% SWE-bench Pro and 69.7% Terminal-Bench 2.0. |
| **Spec** | `/to-spec` | Plan | **Qwen3.7 Max** | Same session, same reasoning requirements. |
| **Tickets** | `/to-tickets` | Plan | **V4 Flash** | Breaking into tickets is mechanical. |
| **Implement** | `/implement` | **Build** | **V4 Pro** | Large-scale code changes need 1M context. |
| **Review** | `/code-review` | Plan | **V4 Flash** | Cheap review. |

---

### 5. PROTOTYPE / SPIKE

Throwaway code to answer a question. Speed > quality.

```
[Prototype] → [Iterate]
```

| Stage | Command | Agent | **Model** | Why |
|---|---|---|---|---|
| **Prototype** | `/prototype` | **Build** | **V4 Flash** or **MiMo V2.5** | Prototype code is throwaway by definition. Use the cheapest model. Both have ~30,000 req/5h = unlimited. MiMo V2.5 gives you 1M context at the same price. |
| **Iterate** | manual | **Build** | Same | |

---

### 6. COMPLEX FEATURE (Wayfinder)

For projects too big for one agent session. Matt's `/wayfinder` skill creates a map of decision tickets (research, prototype, grilling, task).

```
[Chart Map] → [Resolve Tickets One at a Time]
```

| Stage | Command | Agent | **Model** | Why |
|---|---|---|---|---|
| **Chart the map** | `/wayfinder` | Plan | **Qwen3.7 Max** | Naming the destination, surfacing fog, creating the initial tickets. Needs max reasoning. |
| **Research tickets** | (automatic subagent) | Subagent | **V4 Flash** | Reading docs, investigating APIs. High volume, cheap. |
| **Prototype tickets** | `/prototype` | **Build** | **V4 Flash** | Throwaway code. |
| **Grilling tickets** | `/grill-with-docs` | Plan | **K2.7 Code** | Conversation with user to sharpen decisions. |
| **Task tickets** | manual | **Build** | **V4 Flash** | Mechanical setup work. |

---

## Specific Prompt Guidance Per Stage

### `/grill-with-docs` — The Interview

Matt's skill does not interview randomly — it follows a specific pattern:

1. Explore the codebase first (understand current state)
2. Use `/domain-modeling` to challenge fuzzy terms — when you say "user", do you mean the Customer or the Account Holder?
3. Write terms to `CONTEXT.md` as they crystallize
4. Offer ADRs sparingly — only when the decision is:
   - **Hard to reverse**
   - **Surprising without context**
   - **Result of a real trade-off**

**Prompt template:**
```
/grill-with-docs interview me about building a [FEATURE] for [PROJECT].
I want to understand the domain, the problem, and what a good solution looks like.
```

### `/to-spec` — From Conversation to Spec

The skill synthesizes what you already discussed — do NOT re-interview. It produces:

1. **Problem Statement** — from user's perspective
2. **Solution** — from user's perspective  
3. **User Stories** — extensive numbered list (format: "As a <actor>, I want <feature>, so that <benefit>")
4. **Implementation Decisions** — modules, interfaces, schemas, API contracts
5. **Testing Decisions** — seams, what makes a good test for this
6. **Out of Scope** — explicitly what's NOT being built

**Do NOT include file paths or code snippets.** They go stale. Only exception: if a prototype produced a decision-rich snippet (state machine, type shape), inline it with a note that it came from a prototype.

### `/to-tickets` — Breaking into Tickets

Each ticket is a **tracer bullet** — a vertical slice through every layer:
- Schema → API → Logic → Tests → UI (if applicable)
- Each slice is demoable on its own
- Sized to fit a single fresh context window
- Blocking edges declared between tickets

**Prompt to give after spec is ready:**
```
/to-tickets break the spec into vertical-slice tickets and publish them.
```

### `/implement` — Writing the Code

Matt's skill is minimal — it just says "Implement the work described." Key behaviors:
- Uses `/tdd` (test-driven development) where possible, at pre-agreed seams
- Runs typechecking regularly
- Runs single test files regularly
- Runs the full test suite once at end
- Then runs `/code-review` automatically
- Commits to the current branch

### `/code-review` — Two-Axis Review

Runs **two parallel sub-agents**:
- **Standards** — does code follow documented standards + Fowler code smells?
- **Spec** — does code match what the issue/spec asked for?

**Prompt:** Just run `/code-review` and specify the fixed point (commit, branch, or `main`):
```
/code-review review since main
```

---

## Model Route Summary (Quick Reference)

| Task Type | Primary Model | Fallback | Est. req |
|---|---|---|---|
| Triage / categorize | K2.7 Code | V4 Flash | 1-2 |
| Codebase research | K2.7 Code | V4 Flash | 3-8 |
| Planning / spec / arch | Qwen3.7 Max | K2.6 | 1-3 |
| Break into tickets | V4 Flash | K2.7 Code | 3-5 |
| Implementation (simple) | V4 Flash | V4 Pro | 3-8 |
| Implementation (complex) | V4 Pro or K2.6 | Qwen3.7 Max | 5-15 |
| Debugging (loop build) | V4 Flash | K2.7 Code | 10-50 |
| Debugging (hypothesis) | K2.7 Code | K2.6 | 3-5 |
| Prototype | V4 Flash | MiMo V2.5 | 5-20 |
| Code review | V4 Flash | K2.7 Code | 2-4 |

---

## Budget Tracking

$60/month total. A typical feature cycle costs ~$0.50-1.50 depending on complexity.
That's **40-120 features per month** if you route correctly.

| If you use... | Features/month (max) |
|---|---|
| Flash for everything | ~200+ (don't bother — use it for everything) |
| Balanced routing (this guide) | ~60-120 |
| Qwen3.7 Max for everything | ~5-10 (don't do this) |

---

## Learning & Iteration

This workflow is living. As new models arrive on OpenCode Go and skills evolve, update this document. Track changes in git for a history of what changed and why.
