# AI Coding Workflow

Inspired by Matt Pocock's skills — real engineering, not vibe coding.

Uses **OpenCode Go** ($10/mo, $5 first month) with 16 curated open models. All models hosted in US/EU/Singapore, zero-data-retention.

---

## Subscription: OpenCode Go

**Pricing:** $5 first month → $10/month
**Limits per model** (dollar-based, not token-based):

| Window | Budget |
|---|---|
| 5 hours | $12 |
| Weekly | $30 |
| Monthly | $60 |

After limits, falls back to Zen balance (enable in console).

---

## Available Models via OpenCode Go

Sorted by performance tier. All prices per 1M tokens.

### Tier 1 — Maximum Quality (for implementation)

| Model | Input $/1M | Output $/1M | Req/5h | SWE-bench Verif | SWE-bench Pro | Context | Strength |
|---|---|---|---|---|---|---|---|
| **Qwen3.7 Max** | $2.50 | $7.50 | 950 | 80.4% | **60.6%** | 1M | Best SWE-bench Pro score in Go |
| **DeepSeek V4 Pro** | bundled | bundled | 3,450 | 80.6% | 55.4% | 1M | LiveCodeBench 93.5%, Codeforces 3206 |
| **Kimi K2.6** | $0.95 | $4.00 | 1,150 | 80.2% | 58.6% | 256K | Agent Swarm (300 sub-agents), best agentic coding |

### Tier 2 — Best Value Workhorses (for research, review, refactors)

| Model | Input $/1M | Output $/1M | Req/5h | SWE-bench Verif | Context | Strength |
|---|---|---|---|---|---|---|
| **DeepSeek V4 Flash** | $0.14 | $0.28 | **31,650** | 79.0% | 1M | Default for 70-80% of tasks. Cheap, fast. |
| **MiMo-V2.5** | $0.14 | $0.28 | 30,100 | ~78% | 1M | Same price as Flash, 1M context |
| **MiniMax M2.5** | $0.30 | $1.20 | 6,300 | 80.2% | 205K | Best cost-per-benchmark-point |

### Tier 3 — Long-Horizon Autonomy

| Model | Input $/1M | Output $/1M | Req/5h | Strength |
|---|---|---|---|---|
| **MiMo-V2.5-Pro** | $1.74 | $3.48 | 3,250 | Built for multi-hour unsupervized sessions, 1M ctx |
| **GLM-5.1** | $1.40 | $4.40 | 880 | 8h+ autonomous runs (Z.AI reported) |

### Tier 4 — Specialized Picks

| Model | Req/5h | Use case |
|---|---|---|
| Qwen3.6 Plus | 3,300 | Mid-tier Qwen, 1M ctx, good value |
| MiniMax M2.7 | 3,400 | Agentic improvements over M2.5 |
| Kimi K2.5 | 1,850 | Agent Swarm at a discount (100 sub-agents) |

---

## Workflow Stages

### Use Case: Starting from Scratch (New Project)

```
[Interview] → [Spec] → [Tickets] → [Implement] → [Review]
```

| Stage | Command | OpenCode Agent | Recommended Model | Rationale |
|---|---|---|---|---|
| **Interview** | `/grill-with-docs` "interview me about [PROJECT]" | Plan (read-only, Tab to switch) | DeepSeek V4 Flash | Research-heavy, reading docs, asking questions. No code written. Cheapest model handles this fine. 31,650 req/5h = essentially unlimited. |
| **Spec** | `/to-spec` | Plan (read-only) | Qwen3.7 Max or Kimi K2.6 | Spec needs reasoning and structure. Max gives 60.6% SWE-bench Pro — best for planning hard problems. Only ~950 req/5h so use sparingly. |
| **Tickets** | `/to-tickets` | Plan (read-only) | DeepSeek V4 Flash | Breaking spec into tickets is structured and mechanical. Flash is sufficient. Save budget. |
| **Implement** | `/implement` | Build (Tab to switch, has full tool access) | DeepSeek V4 Pro or Kimi K2.6 | This is where actual code gets written. V4 Pro has LiveCodeBench 93.5% and 1M context for large codebases. K2.6 if you need Agent Swarm. 3,450 req/5h for V4 Pro is plenty. |
| **Code Review** | `/code-review` | Plan (read-only) | DeepSeek V4 Flash | Review is reading code, not writing it. Flash at 79.0% SWE-bench Verified is more than enough. Cheap. |

### Use Case: Adding a Feature

```
[Triage] → [Interview] → [Implement] → [Review]
```

| Stage | Command | Agent | Model |
|---|---|---|---|
| **Triage** | `/triage` | Plan | V4 Flash |
| **Interview** | `/grill-with-docs` | Plan | V4 Flash |
| **Implement** | `/implement` | Build | V4 Pro or K2.6 |
| **Review** | `/code-review` | Plan | V4 Flash |

### Use Case: Editing / Fixing a Feature

```
[Diagnose] → [Implement] → [Review]
```

| Stage | Command | Agent | Model |
|---|---|---|---|
| **Diagnose** | `/diagnosing-bugs` | Plan | V4 Flash |
| **Implement** | `/implement` | Build | V4 Pro or K2.6 |
| **Review** | `/code-review` | Plan | V4 Flash |

### Use Case: Architecture Redesign

```
[Architecture] → [Spec] → [Tickets] → [Implement] → [Review]
```

| Stage | Command | Agent | Model |
|---|---|---|---|
| **Architecture** | `/improve-codebase-architecture` | Plan | Qwen3.7 Max (best reasoning) |
| **Spec** | `/to-spec` | Plan | Qwen3.7 Max |
| **Tickets** | `/to-tickets` | Plan | V4 Flash |
| **Implement** | `/implement` | Build | V4 Pro or K2.6 |
| **Review** | `/code-review` | Plan | V4 Flash |

### Use Case: Prototype / Spike

```
[Prototype] → Iterate
```

| Stage | Command | Agent | Model |
|---|---|---|---|
| **Prototype** | `/prototype` | Build | V4 Flash or MiMo-V2.5 (high volume, cheap) |
| **Iterate** | manual | Build | Same |

Skip spec/tickets for throwaway work.

### Use Case: Codebase Design / Planning-Only

```
[Design] → [ADR]
```

| Stage | Command | Agent | Model |
|---|---|---|---|
| **Design** | `/codebase-design` | Plan | Qwen3.7 Max |
| **ADR** | (manually write ADR in `.agents/adr/`) | Plan | V4 Flash |

---

## Model Routing Strategy by Budget

### If you want maximum quality (low volume):
Use **Qwen3.7 Max** for everything planning-related and **DeepSeek V4 Pro** for implementation. ~4,400 combined req/5h. Budget runs out faster but every task uses best-in-class.

### If you want maximum volume (best value):
Use **DeepSeek V4 Flash** as the default for everything. Only escalate to **V4 Pro** or **K2.6** when Flash fails. ~31,650 req/5h on Flash = nearly unlimited daily usage.

### Balanced recommendation (sweet spot):
- **V4 Flash** for interview, tickets, review (~95% of requests)
- **V4 Pro** or **K2.6** for implementation (~4% of requests)
- **Qwen3.7 Max** for spec/architecture on complex projects (~1% of requests)

This stretches your $60/month budget across all use cases while reserving premium models for where they actually matter.

---

## Key Principles

1. **Use the right model for the stage** — don't burn premium tokens on research/review
2. **Tab to switch between agents** — Build (writes code) vs Plan (read-only analysis)
3. **Start fresh sessions per stage** — prevents context bleed (interview context doesn't pollute implementation)
4. **Prototype cheap, implement premium** — throwaway code gets cheap models
5. **Code review is a cheap read** — never use an expensive model for /code-review
6. **If a model fails, escalate up the tier** — Flash → V4 Pro/K2.6 → manual

---

## Model Cost Reference

| Model | Per request (avg) | Monthly requests @ $60 | Cost per feature cycle* |
|---|---|---|---|
| DeepSeek V4 Flash | ~$0.0019 | 158,150 | ~$0.01 |
| MiMo-V2.5 | ~$0.0020 | 150,400 | ~$0.01 |
| DeepSeek V4 Pro | ~$0.0174 | 17,150 | ~$0.10 |
| Kimi K2.6 | ~$0.0522 | 5,750 | ~$0.31 |
| Qwen3.7 Max | ~$0.0632 | 4,770 | ~$0.38 |

*\*Assumes ~5 requests per feature cycle (interview, spec, tickets, implement, review)*

---

## Learning & Iteration

This workflow is living. As new models arrive on OpenCode Go and skills evolve, update this document. Track changes in git for a history of what changed and why.
