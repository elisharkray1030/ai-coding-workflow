# AI Coding Workflow

Inspired by **Matt Pocock's** skills -- AI Engineering/ Coding??? lol.

Uses **OpenCode Go**.
All skills from [`mattpocock/skills`](https://github.com/mattpocock/skills).
0810: reorganized around the grilling-first main pipeline.

---

## The Main Pipeline

```
idea → grilling → to-spec → to-tickets → triage → implement → code-review
```

```mermaid
graph LR
    ID[Idea] --> GR[ /grilling · V4 Flash]
    GR --> SP[ /to-spec]
    SP --> TK[ /to-tickets]
    TK --> TR[ /triage]
    TR -->|ready-for-agent| IM[ /implement · V4 Flash strictly]
    IM --> Q{Quality gate}
    Q -->|Pass| CR[ /code-review]
    Q -->|Fail| IM
    CR --> DN[Done]

    classDef red fill:#ff8787,color:#000
    classDef purple fill:#9775fa,color:#000
    classDef green fill:#69db7c,color:#000
    classDef blue fill:#4dabf7,color:#000
    classDef orange fill:#ffa94d,color:#000
    classDef teal fill:#63e6be,color:#000

    class ID red
    class GR,SP purple
    class TK,TR blue
    class IM orange
    class CR teal
    class DN green
```

Every fuzzy idea enters through **grilling** — an interview in rounds that sharpens it until there are no silent assumptions left. **to-spec** writes it down (no re-interview). **to-tickets** slices it into tracer bullets. **triage** classifies the tickets — and any external issues/PRs — the `ready-for-agent` ones are grabbable by an agent. **implement** builds one. **code-review** gates it on two axes. Triage's other states exit the pipeline: `ready-for-human` / `wontfix` leave it, `needs-info` loops back to the reporter.

---

## Agents

Uses the **`build`** agent via `opencode run --agent build` for everything. It has full tool access (read, edit, bash, subagents) and can handle every stage — planning docs, writing code, spawning research subagents, everything.

The **`plan`** agent exists but is too restricted for this skill set: it can't write CONTEXT.md, ADRs, ticket files, or spawn `general` subagents for delegated work. So `build` is the default for all stages.

---

## Stages

### grilling — the front door

Any fuzzy idea starts here. An interview in rounds that maps the design as a **design tree** — every decision branches into the decisions that hang off it.

- **Rounds** — each round asks the whole **frontier**: every decision whose prerequisites are already settled. Questions you can ask *now* without guessing at answers you haven't heard yet.
- **Numbered + recommended** — each question is numbered with a recommended answer (❓ Q1 + ➡️ my recommendation). You answer; the tree reshapes; the frontier advances. Questions whose answer depends on another open question belong to a later round.
- **Facts are the agent's job** — when a frontier question needs a fact from the environment (filesystem, tools), the agent dispatches a sub-agent to find it. Never asks you for anything it could look up itself. A running exploration is an unsettled prerequisite — only the questions downstream of it wait.
- **Done** — when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Nothing is acted on until you confirm shared understanding.

Variants:
- **grill-me** — plain alias for `/grilling` (runs the same session).
- **grill-with-docs** — same interview + `/domain-modeling`: writes ADRs and the CONTEXT.md glossary as decisions land. Use when you're changing the domain model, not just exploring it.

| Default | Escalation |
|---------|------------|
| V4 Flash | GLM 5.2 or Qwen3.8 Max if the grill comes back shallow (fuzzy CONTEXT.md, unresolved domain terms) |

### to-spec — from conversation to spec

Synthesizes what you already discussed — **do NOT re-interview**. It just writes.

1. Explores the repo, using the domain glossary vocabulary and respecting ADRs in the area it touches.
2. **Sketches the seams first** — where the feature will be tested. Prefers existing seams, highest possible, ideally one. **Checks them with you** before writing anything.
3. Writes the spec: Problem Statement → Solution → extensive numbered User Stories → Implementation Decisions → Testing Decisions → Out of Scope → Further Notes. No file paths or code snippets — they go stale (exception: decision-rich snippets from a prototype).
4. Publishes per the configured tracker (see First-time setup): `.scratch/<feature-slug>/spec.md` on local-markdown, or a real issue on GitHub — and applies **ready-for-agent** (no further triage needed).

| Default | Escalation |
|---------|------------|
| V4 Flash | GLM 5.2 / Qwen3.8 Max if it misses nuance |

### to-tickets — slicing into tracer bullets

Breaks the spec into **tracer-bullet tickets**: narrow vertical slices that each cut through every layer (schema → API → logic → tests → UI) and are demoable on their own. Sized to fit a single fresh context window. Each ticket declares its **blocking edges** — the tickets that must complete before it can start.

- **Wide refactors are the exception** — a mechanical change whose blast radius fans across the whole codebase doesn't slice; it sequences as **expand–contract**: expand (add the new form beside the old, nothing breaks), migrate call sites in batches sized by blast radius (each batch its own ticket, CI stays green because the old form still exists), contract (delete the old form once no caller remains).
- **Quizzes you on granularity** — presents the breakdown (title, blocked-by, what it delivers) and asks: granularity right? blocking edges correct? merge or split? Iterates until you approve.
- **Publishes** — one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered `01`+ in dependency order (blockers first), each with `Status: ready-for-agent`; or native blocking links on a real tracker. Work the **frontier**: any ticket whose blockers are all done.

| Default | Escalation |
|---------|------------|
| V4 Flash | — |

### triage — the state machine gate

Moves issues (and external PRs — **a PR is an issue with attached code**) through a small state machine:

- **Category**: `bug` | `enhancement`
- **State**: `needs-triage` → `needs-info` | `ready-for-agent` | `ready-for-human` | `wontfix`

**ready-for-agent** is the handoff token: fully specified, agent-grabbable — that's the gate into `/implement`. The other states exit the pipeline (`wontfix`, `ready-for-human`) or loop back (`needs-info` → `needs-triage` once the reporter replies).

Process: gather context (redundancy check — already implemented? prior-rejection check — read `.out-of-scope/*.md`) → recommend category + state, wait for direction → **verify the claim** (reproduce the bug from the reporter's steps / check the PR diff and run its tests) → grill if needed → apply the outcome. Every comment posted to the tracker during triage starts with:

```
> *This was generated by AI during triage.*
```

| Default | Escalation |
|---------|------------|
| V4 Flash | Max effort if it consistently misclassifies |

### implement — the build

Implements the work described in the spec or tickets:

- `/tdd` at pre-agreed seams (red → green, one slice at a time)
- Typecheck regularly, single test files regularly
- Full test suite once at the end
- `/code-review` to review the work
- Commit to the current branch

**Strictly V4 Flash.** The highest-volume stage — the only escalation is max effort, never a model switch.

### code-review — two-axis gate

Reviews the diff between `HEAD` and a **fixed point you supply** (commit SHA, branch, tag, `main`, `HEAD~5` — three-dot, so it compares against the merge-base). Runs **two parallel sub-agents**:

- **Standards** — does the code follow the repo's documented coding standards? Always carries the **Fowler smell baseline** (12 smells from *Refactoring* ch.3 — Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, Primitive Obsession, Repeated Switches, Shotgun Surgery, Divergent Change, Speculative Generality, Message Chains, Middle Man, Refused Bequest) on top. Repo standards override the baseline; smells are judgement calls, never hard violations; skip anything tooling already enforces.
- **Spec** — does the code match the originating spec? Found via: issue references in commit messages (`#123`, `Closes #45`) → a path you passed → a spec under `docs/` / `specs/` / `.scratch/` → you. No spec = the Spec axis reports "no spec available".

Reports stay separate — a change can pass one axis and fail the other (standards-following but wrong thing; or exactly what was asked but breaking conventions). No reranking. Ends with a one-line summary per axis.

**Closing the loop** — `/code-review` is the gate, not the end. Discuss any findings it raises, then `/implement` the agreed fixes (strictly Flash) and loop until clean. Tell `/implement` what NOT to fix: a review can flag a "deviation" that is actually correct, and a fix agent will "correct" working code.

| Default | Escalation |
|---------|------------|
| V4 Flash | MiMo V2.5 Pro / MiniMax M3 if the review comes back thin |

---

## Supporting Skills

Skills you reach for along the way — not stages, but called from within them.

- **tdd** — red→green loop; tests at pre-agreed seams only. No horizontal slicing (all tests first = testing imagined behaviour). No tautological or implementation-coupled tests. Expected values must come from an independent source of truth.
- **domain-modeling** — builds the CONTEXT.md glossary + ADRs. Call whenever you're changing the domain model, not just reading it. The engine behind grill-with-docs. ADRs only when **hard to reverse + surprising without context + a real trade-off** — all three, else skip.
- **diagnosing-bugs** — discipline for hard bugs, 6 phases:
  1. **Build a tight red-capable feedback loop** — THE skill. One command that goes red on *this* bug: drives the exact code path, asserts the user's exact symptom, deterministic, fast (seconds), agent-runnable. No loop, no hypothesising — a 30-second flaky loop is barely better than none.
  2. **Reproduce + minimise** — watch it go red on the user's failure (not a nearby one), then shrink the repro until every remaining element is load-bearing.
  3. **Hypothesise** — 3–5 ranked, **falsifiable** hypotheses ("if X is the cause, then changing Y makes it disappear"). Shown to you before testing.
  4. **Instrument** — one variable at a time. Debugger/REPL over logs; tagged debug logs (`[DEBUG-xxxx]`, one grep to clean); never "log everything and grep". Perf bugs: measure and bisect, logs are usually wrong.
  5. **Fix + regression** — regression test before the fix, at a **correct seam** (exercises the real bug pattern at the call site). No correct seam = that itself is the finding — flag it.
  6. **Cleanup + post-mortem** — instrumentation removed, throwaways deleted, the correct hypothesis stated in the commit, then: *what would have prevented this bug?* — hands off to improve-codebase-architecture when the answer is architectural.
- **research** — delegated fact-finding against high-trust sources. Resolves wayfinder research tickets.
- **prototype** — throwaway artifact to answer "how should it look / behave". One command to run, no persistence, no polish, verdict captured and committed to a throwaway branch.
- **handoff** — compacts the conversation into a handoff doc for a fresh agent. Saved to the OS temp dir (not the repo), with a **suggested-skills** section. References other artifacts (specs, ADRs, tickets, commits) instead of duplicating them; redacts secrets.
- **improve-codebase-architecture** — architecture scan for deepening opportunities, visual HTML report using the codebase-design vocabulary (module, interface, depth, seam, adapter, leverage, locality). V4 Flash by default; max effort if the scan comes back shallow. Also the handoff target from diagnosing-bugs phase 6.

---

## Scaled-up: wayfinder

When work is too big for one agent session, **grilling scales up** into wayfinding: a shared **map** (one issue labelled `wayfinder:map`) of **decision tickets** — questions whose resolution is a decision, not slices of a build to execute. The map is an **index, not a store**: it gists each decision and links to the ticket holding the detail. Refer to tickets **by name**, never by bare id.

- **Charting** — a grilling + domain-modeling session names the **destination** (the spec/decision/change this effort is finding its way to), then grills breadth-first across the whole space. No fog surfaced = the whole journey fits one session — don't build a map. Otherwise create the map and the specifiable tickets, wire blocking edges in a second pass, fire the research subagents.
- **Ticket types** — research (AFK, resolved by a `/research` subagent), prototype (HITL), grilling (HITL, the default), task (does rather than decides — unblocks a decision).
- **Claim first** — a session claims a ticket by assigning it to itself before any work, so concurrent sessions skip it.
- **Frontier** — open, unblocked, unclaimed tickets. Blocking uses the tracker's **native** dependency relationship so the frontier renders visually in the tracker's own UI.
- **Fog of war** — in-scope decisions you can see coming but can't yet phrase sharply live in the map's **Not yet specified** section; they graduate into tickets as the frontier advances. Out-of-scope work never graduates — it stays out even if the destination is redrawn.
- **Pace** — never resolve more than one ticket per session (research tickets excepted). A resolution is a comment + close + a one-line context pointer in the map's "Decisions so far".

| Stage | Default | Escalation |
|-------|---------|------------|
| Chart the map | V4 Flash | Max effort only if the initial map is wrong |
| Re-chart | V4 Flash | Max effort (rare — destination shifted or map was wrong) |
| Research / prototype / task tickets | V4 Flash | — |
| Grilling tickets | V4 Flash | GLM 5.2 / Qwen3.8 Max if a session stalls |

---

## Alternate Paths

Not every task runs the full pipeline. Shortcuts:

### New project (full pipeline)
The whole main pipeline end to end. **Est: simple ~$0.04 | complex ~$0.13**

### Adding a feature (trimmed pipeline)
```
idea → (light grilling) → implement → code-review
```
Skip spec and tickets — you're extending what's there. Cross-cutting change (3+ modules)? Rerun `/implement` at max effort — still V4 Flash, no model switch. **Est: simple ~$0.03 | cross-cutting ~$0.08**

### Bug fix
```
diagnosing-bugs → code-review → discuss issues → implement (agreed fixes) → loop until clean
```
Escalation: diagnosing-bugs → GLM 5.2 (max effort) / Qwen3.8 Max if the fix keeps failing; review → MiMo V2.5 Pro / MiniMax M3 if thin; the fix loop stays strictly Flash. **Est: easy ~$0.05 | hard ~$0.18**

### Architecture redesign
```
improve-codebase-architecture → to-spec → to-tickets → implement → code-review
```
The scan produces an HTML report of deepening opportunities; `/to-spec` formalizes the plan from it. **Est: light ~$0.08 | deep ~$0.25**

### Prototype / spike
```
prototype → iterate → ... until answered
```
Throwaway code, the cheapest loop in the book. V4 Flash or MiMo V2.5 — speed over quality; GLM 5.2 only if the design question resists. No spec, no review — this is learning, not shipping. **Est: ~$0.01**

---

## First-time setup (per repo)

Before the engineering skills work, run once per repo: `/setup-matt-pocock-skills`

- **Issue tracker** — GitHub (gh CLI), GitLab (glab CLI), or **local-markdown** (`.scratch/<feature>/` — good for solo projects or repos without a remote)
- **Triage labels** — defaults are the five canonical role names (recommended); overrides only if the tracker already uses different strings
- **Domain docs** — single-context: root `CONTEXT.md` + `docs/adr/` (default, fits almost every repo). Multi-context (`CONTEXT-MAP.md`) only for real monorepos.

Writes `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`, `docs/agents/domain.md`, and an `## Agent skills` block in AGENTS.md / CLAUDE.md. Edit those files directly later — re-run only to switch trackers or restart.

---

## Routing Feedback

Escalations are data. If a task type keeps requiring escalation, the routing table should evolve.

**Track escalations per task type.** After each project or sprint, note which tasks escalated and to which effort level:

```
Task: cross-cutting feature-add to auth module
Escalated: /implement → max effort (structural failure)
Pattern: 3rd time in 2 weeks
Action: bump default for auth-scoped work to start at max effort
```

**When to update the routing table:**
- 3+ escalations of the same type in 2 weeks → change the default effort for that task type
- A model you're routing to gets deprecated or replaced → update immediately
- A new model at Flash prices outperforms Flash on your workload → swap the default
- A new model makes max effort obsolete → adopt it as the new default

**Keep it live.** This is a workflow doc — stale routing is worse than no routing. If a model gets better or cheaper, the defaults here should follow. The escalation table at the top is the first thing to touch when patterns emerge.

---

<details>
<summary><strong>Routing & Models</strong> — click to expand</summary>

## Routing Philosophy

**V4 Flash is the default for everything.** It scores 79% SWE-bench Verified, has 1M context, costs $0.14/$0.28 per million tokens, and has 31,650 req/5h — effectively unlimited. Since the 0731 update it handles *every* stage of *every* pipeline — including architecture scans and wayfinding that used to be pinned to pricier models.

**Escalate only when Flash proves insufficient.** Since the 0805 update, escalation is per-stage: the thinking stages step up to a premium model, while the high-volume code-writing stages stay strictly on Flash. Don't pre-assign expensive models to stages based on what the stage *could* need — wait for a concrete failure, then rerun that stage on its escalation target.

| Stage | Escalate to |
|-------|-------------|
| grilling, to-spec | GLM 5.2 or Qwen3.8 Max |
| prototype | GLM 5.2 |
| implement, tdd | **V4 Flash, strictly** — max effort only, never a model switch |
| code-review | MiMo V2.5 Pro or MiniMax M3 |
| diagnosing-bugs | GLM 5.2 (max effort) or Qwen3.8 Max |
| Everything else | Max effort on V4 Flash (unchanged) |

Rule of thumb: the volume stages (implement, tdd) burn the most tokens — keep them on Flash. Spend escalations on the thinking stages (grilling, spec, debugging, review), where a smarter model pays for itself.

The savings are dramatic: ~$0.04 for a simple feature vs ~$0.42 with the old model-per-stage routing. You stay safely within the $60/month budget even on heavy months.

## Model Reference

Pricing via OpenCode Go. "Req/5h" = estimated requests per 5-hour rolling window.

| Model | Input $/1M | Output $/1M | Req/5h | Req/mo | Context | Key Strength |
|---|---|---|---|---|---|---|
| **Qwen3.7 Max** | $2.50 | $7.50 | 950 | 4,770 | 1M | Highest SWE-bench Pro on Go (60.6%). Best for hard planning. |
| **Qwen3.8 Max** | $2.00 | $6.00 | — | — | 1M | New Aug 2026. Multimodal (text/image/video). Escalation target for grill/spec/debugging. |
| **DeepSeek V4 Pro** | $0.435 | $0.87 | 3,450 | 17,150 | 1M | LiveCodeBench 93.5%, Codeforces 3206. Strongest for implementation. |
| **Kimi K2.6** | $0.95 | $4.00 | 1,150 | 5,750 | 262K | Agent Swarm (300 sub-agents). Best for agentic multi-file changes. |
| **Kimi K2.7 Code** | $0.95 | $4.00 | 1,350 | 6,750 | 256K | Coding-focused model. More requests than K2.6. Solid mid-tier planner. |
| **DeepSeek V4 Flash** | $0.14 | $0.28 | **31,650** | 158,150 | 1M | **Default workhorse.** 79% SWE-bench Verified. Cheap. Fast. |
| **MiMo V2.5** | $0.14 | $0.28 | 30,100 | 150,400 | 1M | Budget workhorse. Same price as Flash, 1M context. |
| **MiniMax M2.7** | $0.30 | $1.20 | 3,400 | 17,000 | 205K | Strong cost-per-benchmark-point (78% SWE-bench Verified at $0.30). |
| **Grok 4.5** | $2.00 | $6.00 | 120 | 600 | 1M | xAI's latest. Fast reasoning, large context. |
| **GLM-5.2** | $1.40 | $4.40 | 880 | 4,300 | 1M | Zhipu flagship. Strong bilingual coding (CN/EN). |
| **GLM-5.1** | $1.40 | $4.40 | 880 | 4,300 | 128K | Solid all-rounder from Zhipu. |
| **Kimi K3** | $3.00 | $15.00 | 110 | 490 | 128K | Moonshot's coding specialist. High output cost — use sparingly. |
| **MiMo V2.5 Pro** | $0.435 | $0.87 | 3,250 | 16,300 | 1M | Upgraded MiMo. Same price tier as V4 Pro, lower request cap. |
| **MiniMax M3** | $0.30 | $1.20 | 3,200 | 16,000 | 1M | MiniMax's latest. Improved over M2.7 at same price. |
| **Qwen3.7 Plus** | $0.40 | $1.60 | 4,300 | 21,600 | 1M | Strong mid-tier Qwen. Good balance of cost and quality. |
| **Qwen3.6 Plus** | $0.50 | $3.00 | 3,300 | 16,300 | 256K | Earlier Qwen gen at mid-range pricing. Solid reasoning. |
| **Hy3** | $0.14 | $0.58 | 4,300 | 21,500 | 128K | Budget model. High throughput at Flash-like input pricing. |

## Model Route Quick Reference

| Task Type | Default | Escalation | Agent | Est. req |
|-----------|---------|------------|-------|----------|
| Triage | V4 Flash | Max effort | build | 1-2 |
| Grilling (incl. grill-with-docs) | V4 Flash | GLM 5.2 / Qwen3.8 Max if shallow | build | 3-8 |
| Planning / spec (new project) | V4 Flash | GLM 5.2 / Qwen3.8 Max | build | 1-3 |
| Tickets | V4 Flash | — | build | 3-5 |
| Implementation (simple) | V4 Flash | — | build | 3-8 |
| Implementation (complex) | V4 Flash | Max effort only — no model switch | build | 5-15 |
| Debugging | V4 Flash | GLM 5.2 (max effort) / Qwen3.8 Max | build | 10-50 |
| Architecture scan (light) | V4 Flash | Max effort if shallow | build | 1-2 |
| Architecture scan (deep w/ grill loop) | V4 Flash | Max effort if shallow | build | 3-6 |
| Prototype | V4 Flash / MiMo | GLM 5.2 | build | 5-20 |
| Code review | V4 Flash | MiMo V2.5 Pro / MiniMax M3 | build | 2-4 |
| Research | V4 Flash | — | build | 2-5 |
| Handoff | V4 Flash | — | build | 1 |
| Wayfinder (map) | V4 Flash | Max effort if map is wrong | build | 1-3 |
| Wayfinder (re-chart) | V4 Flash | Max effort (rare) | build | 1-2 |
| Wayfinder (tickets) | V4 Flash | — | build | 3-10+ |

</details>

---

<details>
<summary><strong>Budget Tracking</strong> — click to expand</summary>

$60/month. A typical feature cycle costs ~**$0.04-0.25** with the Flash-for-everything routing.
That's **240-1,500 features per month** if you route correctly.

| Routing strategy | Features/month (max) |
|---|---|
| **Flash for everything (this guide)** | **~240-1,500** |
| Balanced (per-stage model pinning, pre-0731) | ~60-120 |
| Max effort on every call | ~200+ (slower, same token cost) |

Escalation to max effort costs the same per token as normal Flash — the only price is latency, not dollars. Model escalations (GLM 5.2 $1.40/$4.40, Qwen3.8 Max $2.00/$6.00) cost more per token but are rare by design — they're the exception, not the default. The buffer is large enough ($40-50/month) that even heavy months with multiple architecture scans, wayfinders, and bug fixes won't break the budget.

</details>

---

## Learning & Iteration

This is for me to document my workflow plan so things will change over time~. Models on Go... tools I have access too.. local models??! new models??! subscriptions??!.

0810 update: reorganized around the grilling-first main pipeline (idea → grilling → to-spec → to-tickets → triage → implement → code-review). Triage moved from the front door to the gate before implement; wayfinder reframed as scaled-up grilling; new supporting skills documented (handoff, research); the `.scratch/` tracker convention and `/setup-matt-pocock-skills` per-repo setup added. All stage mechanics verified against `mattpocock/skills` @ main (Aug 2026).

0805 update: escalation is now per-stage model switching — thinking stages step up (GLM 5.2, Qwen3.8 Max, MiMo V2.5 Pro, MiniMax M3), /implement stays strictly on V4 Flash. Open question: do any stages deserve a model above the current escalation targets (gpt-5.6-luna just landed on Go)? Revisit as the lineup grows.

Also in upstream, not yet adopted: to-questionnaire, wait-what, ask-matt, teach, wizard, writing-for-agents.
