# Models

Pricing, request limits, and routing for OpenCode Go. Escalations are data — see Routing feedback below for when to update this page. Updated Aug 24 2026 against [official Go docs](https://opencode.ai/docs/go/).

---

<details>
<summary><strong>Routing & models</strong> — expand</summary>

Rule of thumb: volume stages (implement, tdd) burn the most tokens, so they stay on Flash. Spend escalations on the thinking stages (grilling, spec, debugging, review). A simple feature costs ~$0.08 on Flash at peak vs ~$0.50 with per-stage model routing.

**Model Reference** (pricing via OpenCode Go, updated Aug 26 2026; DeepSeek has peak/off-peak — HK work hours are peak. Per-model usage caps apply — see Budget column.)

| Model | Input $/1M | Output $/1M | Req/mo | Budget | Context | Key Strength |
|---|---|---|---|---|---|---|
| **MiMo V2.5** | $0.14 | $0.28 | 150,400 | $60 | 1M | **Budget king.** Cheapest per-token on Go. |
| **Hy3** | $0.14 | $0.58 | 21,500 | $60 | 128K | Budget model. Flash-like input pricing, higher output cost. |
| **DeepSeek V4 Flash** | $0.22–0.44 | $0.66–1.32 | 37,800 | $30 | 1M | **Default workhorse.** 79% SWE-bench Verified. Peak pricing during HK work hours. |
| **MiniMax M2.5** | $0.30 | $1.20 | 17,000 | $60 | 1M | Legacy. Same price as M2.7 — prefer M2.7 or M3. |
| **MiniMax M2.7** | $0.30 | $1.20 | 17,000 | $60 | 1M | Strong cost-per-benchmark-point (78% SWE-bench Verified). |
| **MiniMax M3** | $0.30 | $1.20 | 16,000 | $60 | 1M | MiniMax's latest. Good review escalation value. |
| **GLM-5.1** | $1.40 | $4.40 | 4,300 | $60 | 128K | Solid all-rounder from Zhipu. |
| **GLM-5.2** | $1.40 | $4.40 | 4,300 | $60 | 1M | Zhipu flagship. Strong bilingual coding (CN/EN). Best escalation value. |
| **GLM-5.3** | $1.40 | $4.40 | 1,080 | $15 | 1M | Same price as GLM-5.2 but fewer requests — not worth routing to. |
| **GPT 5.6 Luna** | $0.20–0.40 | $1.20–1.80 | 10,250 | $15 | 1.05M | Cost champion for agentic code. Weak long-context recall (MRCR 41%). >272K tokens doubles price. |
| **Kimi K2.6** | $0.95 | $4.00 | 5,750 | $60 | 262K | Agent Swarm (300 sub-agents). Agentic multi-file changes. |
| **Kimi K2.7 Code** | $0.95 | $4.00 | 6,750 | $60 | 256K | Coding-focused. More requests than K2.6. Solid mid-tier planner. |
| **MiMo V2.5 Pro** | $0.435 | $0.87 | 16,300 | $15 | 1M | Upgraded MiMo. Same input price as V4 Pro, better output cost. |
| **DeepSeek V4 Pro** | $0.66–1.32 | $1.98–3.96 | 5,200 | $15 | 1M | LiveCodeBench 93.5%. Strong but expensive — peak pricing in HK hours. |
| **Qwen3.7 Plus** | $0.40–1.20 | $1.60–4.80 | 21,600 | $60 | 1M | Strong mid-tier Qwen. >256K tokens triples price. |
| **Qwen3.6 Plus** | $0.50–2.00 | $3.00–6.00 | 16,300 | $60 | 256K | Earlier Qwen gen. >256K tokens quadruples price. |
| **Qwen3.7 Max** | $2.50 | $7.50 | 1,690 | $60 | 1M | Highest SWE-bench Pro on Go (60.6%). Best for hard planning. |
| **Qwen3.8 Max** | $2.00 | $6.00 | 810 | $15 | 1M | Multimodal. Last-resort second opinion only — no published benchmarks. |
| **Grok 4.5** | $2.00 | $6.00 | 600 | $15 | 1M | xAI's latest. Fast reasoning. Very limited requests. |
| **Kimi K3** | $3.00 | $15.00 | 490 | $15 | 128K | Moonshot's coding specialist. High output cost — use sparingly. |

**Model Route Quick Reference**

| Task Type | Model | Agent | Est. req |
|-----------|-------|-------|----------|
| Triage | V4 Flash<br>max effort | build | 1-2 |
| Grilling (incl. grill-with-docs) | V4 Flash<br>GLM 5.2 / Luna if shallow | build | 3-8 |
| Planning / spec (new project) | V4 Flash<br>Luna if it misses nuance | build | 1-3 |
| Tickets | V4 Flash<br>Luna if wrong-sized | build | 3-5 |
| Implementation (simple) | V4 Flash<br>— | build | 3-8 |
| Implementation (complex) | V4 Flash<br>max effort only — no model switch | build | 5-15 |
| Debugging | V4 Flash<br>GLM 5.2 (max effort) / Luna | build | 10-50 |
| Architecture scan (light) | V4 Flash<br>max effort if shallow | build | 1-2 |
| Architecture scan (deep w/ grill loop) | V4 Flash<br>max effort if shallow | build | 3-6 |
| Prototype | V4 Flash / MiMo<br>Luna | build | 5-20 |
| Code review | V4 Flash<br>MiMo V2.5 Pro / MiniMax M3 | build | 2-4 |
| Research | V4 Flash<br>GLM 5.2 if thin | build | 2-5 |
| Handoff | V4 Flash<br>GLM 5.2 (max effort) if it loses context | build | 1 |
| Wayfinder (map) | V4 Flash<br>max effort if map is wrong | build | 1-3 |
| Wayfinder (re-chart) | V4 Flash<br>max effort (rare) | build | 1-2 |
| Wayfinder (tickets) | V4 Flash<br>— | build | 3-10+ |

</details>

---

<details>
<summary><strong>Budget tracking</strong> — expand</summary>

$10/month subscription gives per-model usage caps ($15–$60 per model depending on tier). A typical feature cycle costs ~**$0.04–0.25** → **240–1,500 features/month** if you route correctly. When a model's cap is hit, you can still use free models or fall back to Zen balance.

| Routing strategy | Features/month (max) |
|---|---|
| **Flash for everything (this guide)** | **~240–1,500** |
| Balanced (per-stage model pinning, pre-0731) | ~60–120 |
| Max effort on every call | ~200+ (slower, same token cost) |

Max-effort escalation costs the same per token, the only price is latency. Model escalations (GLM 5.2, Luna) cost more per token but are rare by design. The $40–50 buffer covers even heavy months (multiple architecture scans, wayfinders, bug fixes).

</details>

---

## Routing feedback

Escalations are data. Track them per task type, and update the tables when: 3+ same-type escalations in 2 weeks, a routed model gets deprecated, or a model at Flash prices beats Flash. Stale routing is worse than no routing.


