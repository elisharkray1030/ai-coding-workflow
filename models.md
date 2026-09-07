# Models

Full pricing, request limits, and model list live in the [official Go docs](https://opencode.ai/docs/go/). This file tracks routing decisions and budget patterns — things that change slowly.

---

<details>
<summary><strong>Routing & models</strong> — expand</summary>

Rule of thumb: volume stages (implement, tdd) burn the most tokens, so they stay on the default model. Spend escalations on the thinking stages (grilling, spec, debugging, review). A simple feature costs ~$0.08 on the default at peak vs ~$0.50 with per-stage model routing.

**Current defaults:**
- **Primary workhorse:** Muse Spark 1.3 — replaces DeepSeek V4 Flash (limits slashed)
- **Escalation for thinking stages:** GLM 5.2 (grilling, diagnosing-bugs) / Luna (spec, tickets, prototype)
- **Escalation for code review:** MiMo V2.5 Pro / MiniMax M3
- **Code stages:** primary model only, max effort is the only escalation

**Model Route Quick Reference**

| Task Type | Model | Agent | Est. req |
|-----------|-------|-------|----------|
| Triage | primary<br>max effort | build | 1-2 |
| Grilling (incl. grill-with-docs) | primary<br>GLM 5.2 / Luna if shallow | build | 3-8 |
| Planning / spec (new project) | primary<br>Luna if it misses nuance | build | 1-3 |
| Tickets | primary<br>Luna if wrong-sized | build | 3-5 |
| Implementation (simple) | primary<br>— | build | 3-8 |
| Implementation (complex) | primary<br>max effort only — no model switch | build | 5-15 |
| Debugging | primary<br>GLM 5.2 (max effort) / Luna | build | 10-50 |
| Architecture scan (light) | primary<br>max effort if shallow | build | 1-2 |
| Architecture scan (deep w/ grill loop) | primary<br>max effort if shallow | build | 3-6 |
| Prototype | primary / MiMo<br>Luna | build | 5-20 |
| Code review | primary<br>MiMo V2.5 Pro / MiniMax M3 | build | 2-4 |
| Research | primary<br>GLM 5.2 if thin | build | 2-5 |
| Handoff | primary<br>GLM 5.2 (max effort) if it loses context | build | 1 |
| Wayfinder (map) | primary<br>max effort if map is wrong | build | 1-3 |
| Wayfinder (re-chart) | primary<br>max effort (rare) | build | 1-2 |
| Wayfinder (tickets) | primary<br>— | build | 3-10+ |

</details>

---

## Routing feedback

Escalations are data. Track them per task type, and update the tables when: 3+ same-type escalations in 2 weeks, a routed model gets deprecated, or a model at default prices beats the current default. Stale routing is worse than no routing.
