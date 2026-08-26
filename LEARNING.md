# Learning & iteration

This is for me to document my workflow plan so things will change over time~. Models on Go... tools I have access too.. local models??! new models??! subscriptions??!.

<details>
<summary>0826: GLM-5.3 = ox alpha reveal + per-model budget caps</summary>

- ox alpha revealed as GLM-5.3 Flash — same $1.40/$4.40 pricing as GLM-5.2 but $15 budget cap (vs $60 for GLM-5.2) means ~1,080 req/mo vs 4,300. Not worth routing to — GLM-5.2 is strictly better value
- Per-model usage caps now explicit in docs: most models are $60, but premium/limited models (GLM-5.3, Luna, MiMo V2.5 Pro, V4 Pro, Qwen3.8 Max, Grok 4.5, Kimi K3) are $15. V4 Flash is $30
- Muse Spark 1.2 Contrib dropped from Go docs — removed from reference table
- Budget section updated: $10/mo subscription with per-model caps, not a flat $60 pool

</details>

<details>
<summary>0820: Luna for bounded thinking stages</summary>

- /to-spec and /to-tickets escalation shifted from GLM 5.2 → Luna
- Rationale: spec and tickets are bounded single calls (10-30K context), Luna's MRCR cliff irrelevant; 7x cheaper input, 2.4x more requests
- /grilling stays GLM 5.2 — design tree accumulates across rounds, recall critical, 84s Luna latency kills interview flow
- /diagnosing-bugs stays GLM 5.2 — debugging needs hypothesis tracking across turns
- /implement stays Flash — 79% SWE-bench Verified, proven at scale
- Pattern: GLM for long-context thinking (grilling, debugging), Luna for bounded calls (spec, tickets), Flash for code

</details>

<details>
<summary>0820: OpenCode Go docs refresh</summary>

- DeepSeek V4 Flash/Pro now have peak/off-peak pricing — HK work hours (9am-6pm HKT = 01-10 UTC) are peak, so Flash input is $0.44 (not the old $0.14)
- Request limits slashed across the board: Flash 31,650→7,600/5h, V4 Pro 3,450→1,050/5h, Qwen3.7 Max 950→340/5h
- MiMo V2.5 ($0.14/$0.28) is now the real budget king — Flash got more expensive while MiMo stayed the same
- New models: GLM-5.3 ($15 budget, fewer requests than GLM-5.2 — not worth routing to), MiniMax M2.5 (same as M2.7)
- Qwen3.8 Max now has data: 160 req/5h, 810/mo — still last-resort only
- Tiered pricing added: Luna >272K doubles, Qwen3.7 Plus >256K triples, Qwen3.6 Plus >256K quadruples
- MiniMax M2.7 context updated 205K→1M
- Model table re-sorted cheapest-first by effective cost
- Routing strategy unchanged — Flash is still cheapest per-token, escalation targets stay the same

</details>

<details>
<summary>0812: triage is the inbound lane, not a pipeline gate</summary>

- upstream (triage SKILL.md @ main + aihero.dev/skills-triage): "/triage is only for issues you didn't create"; to-tickets output is ready-for-agent by construction; triage is the periodic maintenance pass at the front of the tracker, upstream of the build chain
- the 0810 "triage to the gate before implement" move was a misread — reverted; main chain is now idea → grilling → to-spec → to-tickets → implement → code-review
- triage kept as the inbound lane for external issues/PRs, with the full mechanics: verify before brief, agent brief = the contract (AGENT-BRIEF.md), .out-of-scope concept files (already-implemented never filed), needs-info template, quick override, labels created by hand (#616)

</details>

<details>
<summary>0812: Luna + GLM routing decision</summary>

- prototype escalation → Luna (7x cheaper input, faster gen, image input for UI)
- GLM 5.2 keeps grilling/to-spec/diagnosing-bugs — Luna's MRCR ~41% recall cliff + ~84s max-effort latency = wrong tool for interview/synthesis/debug loops
- Qwen3.8 Max dropped from escalation lines ($2/$6, 810 req/mo, no published benchmarks) — last-resort second opinion only
- /implement still strictly Flash

</details>

<details>
<summary>0811: review built into implement</summary>

- /code-review auto-runs at the end of implement; standalone stage only for on-demand range reviews
- after a ticket ships, ask if anything else needs fixing/building → loop from to-spec
- start /implement in a fresh session (spec + tickets read from `.scratch/<slug>/`)

</details>

<details>
<summary>0810: grilling-first pipeline</summary>

- triage moved from front door to the gate before implement (reverted 0812 — triage is the inbound lane, not a pipeline step); wayfinder = grilling at scale
- added handoff/research, the `.scratch/` convention, and per-repo setup
- mechanics verified against `mattpocock/skills` @ main (Aug 2026)

</details>

<details>
<summary>0805: per-stage escalation (open question → answered 0812)</summary>

- thinking stages step up (GLM 5.2, Qwen3.8 Max, MiMo V2.5 Pro, MiniMax M3); /implement stays strictly Flash
- open question ("does any stage deserve a model above the escalation targets?") — resolved 0812: Luna adopted for /prototype only

</details>

Also in upstream, not yet adopted: to-questionnaire, wait-what, ask-matt, teach, wizard, writing-for-agents.
