# AI Coding Workflow

Inspired by **Matt Pocock's** skills -- AI Engineering/ Coding??? lol. Runs on **OpenCode Go** with [`mattpocock/skills`](https://github.com/mattpocock/skills).

**Contents:** [Setup](#setup) · [Agents](#agents) · [The main pipeline](#the-main-pipeline) · [Alternate paths](#alternate-paths) · [skills.md](skills.md) · [models.md](models.md) · [LEARNING.md](LEARNING.md)

---

## Setup

Run `/setup-matt-pocock-skills` once per repo: pick a tracker (GitHub / GitLab / local `.scratch/`), triage labels, domain docs. It writes `docs/agents/*.md` plus an `## Agent skills` block in AGENTS.md / CLAUDE.md.
- Create the 7 triage labels once per tracker (`gh label create bug enhancement needs-triage needs-info ready-for-agent ready-for-human wontfix`) — setup writes the mapping, not the labels (#616)

## Agents

Every stage runs on the **build** agent. Mid-stage: flip to `plan` to discuss what you want, back to `build` to execute.

---

## The main pipeline

```
idea → grilling → to-spec → to-tickets → implement → code-review
```

```mermaid
graph LR
    ID[Idea] --> GR[ /grilling]
    GR --> SP[ /to-spec]
    SP --> TK[ /to-tickets]
    GR -->|"fits one window"| IM[ /implement · Muse Spark 1.3 strictly]
    TK -->|"multi-session"| IM
    IM --> Q{Quality gate}
    Q -->|Pass| CR[ /code-review]
    Q -->|Fail| IM
    CR --> LOOP{Anything else to fix or build?}
    LOOP -->|More to build or fix| SP
    LOOP -->|Nothing left| DN[Done]

    EXT[/External issue or PR/] --> TR[ /triage]
    TR -->|agent brief| IM

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

A fuzzy idea walks in, gets grilled into shape, then built and reviewed before it ships. How far down the chain it goes depends on size — see below. Work that arrives from outside — bug reports, feature requests, unannounced PRs — enters through the triage lane (see [skills.md](skills.md)): verified, briefed, then dropped into the same implement queue.

### Pick by build size

One question picks the depth: **can the build fit in one fresh context window?**

| Build | Flow | Example |
|---|---|---|
| tiny — could've just done it | grill → implement (same window) | add a flag, accept single-digit hours |
| small — decisions worth keeping | grill → to-spec → implement (fresh session, reads spec) | a name-matching rule worth recording |
| multi-session / parallel slices | full chain, `/implement` per ticket | fete POS: skeleton → cash sale → tenders → EOD |

- grill → to-spec → to-tickets stay in one unbroken window; every `/implement` starts fresh
- too-low tell: implement keeps blowing its window · too-high tell: the spec has one obvious ticket

---

## Alternate paths

| Path | Flow | Est. cost |
|------|------|-----------|
| New project | full pipeline | ~$0.04–0.13 |
| Small feature | grill → spec → implement (build fits one window — [pick by build size](#pick-by-build-size)) | ~$0.02–0.05 |
| Feature add | pick depth by [build size](#pick-by-build-size); cross-cutting: implement at max effort | ~$0.03–0.08 |
| Bug fix | diagnosing-bugs → review → discuss → implement → loop | ~$0.05–0.18 |
| Arch redesign | scan → spec → tickets → implement → review | ~$0.08–0.25 |
| Prototype | prototype → iterate | ~$0.01 |

---

Every skill — stages, supporting skills, triage, wayfinder — lives in [skills.md](skills.md), with per-skill Default/Escalation routing lines. Model pricing, request limits, and budget tracking are in [models.md](models.md). Workflow decisions over time: [LEARNING.md](LEARNING.md).
