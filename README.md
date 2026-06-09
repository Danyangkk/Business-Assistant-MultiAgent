# Business-Assistant · Merchant Assistant AI QA Platform

> A **frontend prototype** of an AI QA bot for enterprise-WeChat customer group chats, built on an **orchestrator-workers** architecture (one orchestrator + five workers). It isn't a running system — it turns the orchestration, slot-filling, silent-intent, full trace, and quality dashboard into a **clickable UI**, so an architecture spec becomes something you can demo and argue over. **Single file, zero backend, runs offline.**

![Type](https://img.shields.io/badge/type-frontend%20prototype-204eff.svg)
![Single file](https://img.shields.io/badge/single--file-HTML%2FCSS%2FJS-16a34a.svg)
![No backend](https://img.shields.io/badge/backend-none-lightgrey.svg)
![Architecture](https://img.shields.io/badge/agent-orchestrator--workers-purple.svg)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

[中文版](./README.zh-CN.md) | **English**

---

## What it is

A product prototype for an AI QA bot living in **enterprise-WeChat customer group chats**. The bot stands in for ops staff and guides brand partners through a merchant back-office system — a partner drops a question in the chat ("Where's my shipment?", "How do I create a guaranteed-volume contract?", "Was I penalized this month?") and the bot answers with both fetched data and operations-manual evidence.

At its core is an **orchestrator-workers** setup (inspired by Anthropic's multi-agent research pattern): retrieval is just one worker, and an orchestrator handles task decomposition, routing, parallel dispatch, replanning, and the final synthesis. This repo is its **frontend**: eight pages plus a handful of overlays, all backed by hard-coded mock data, turning every abstract piece of the architecture into something you can see and click.

The online answering pipeline looks like this:

```
Group message arrives
   ↓
Ingress        Resolve business unit + brand (context = BU · brand · group-id, carried end-to-end)
   ↓
Preprocess     Clean · last 5 turns · pending check
   ↓
Intent gate    need_reply? ──no──→ stay silent (nothing posted, just a UI log)
   ↓ yes
Fast path      Cache hit / FAQ hit ──hit──→ answer directly, skip the orchestrator
   ↓ miss
Orchestrator   Split subtasks → route by worker description → dispatch in parallel → ReAct replan (≤5 rounds)
   ├─ RAG worker              ops-manual KB / this group's conversation archive (orchestrator decides whether to add the archive source)
   ├─ logistics / contract / quote / penalty   fetch via the merchant-system MCP (BU + brand auth)
   └─ worker missing a param? → back to orchestrator, which asks the group once (slot-filling, pending 3h)
   ↓
Synthesis      Collect each worker's distilled result → LLM generates the final reply
   ↓
Post-process & send   + full trace persisted
```

## Features

- **🧩 Orchestration, made visible** — task decomposition, five workers running in parallel, ReAct replanning, final synthesis: every step shows up in the UI, not a black box that just spits out one line.
- **👀 One bot account, two views** — the tech lead sees the **full trace** + quality dashboard; the business lead sees only each worker's **assembled context**. Same conversation, different role, different visibility — flip the role and it changes instantly.
- **🔍 RAG returns evidence, not just an answer** — expand a worker tag **inline** to see retrieved chunks (top-5, with rerank scores, already filtered below 0.7); click a chunk index to **jump straight into the matching document** in the knowledge base and highlight that passage.
- **🔇 Stays quiet when it should** — messages the intent gate marks "no reply needed" are never posted to the group, only logged in the backend; in the effect-test feed, silent conversations are their own distinct state, not mixed in with answered ones.
- **🧱 Three-axis isolation, throughout the UI** — business unit (ops manual + business data) / group-chat id (conversation archive) / brand (data-fetch auth): all three are filterable and attributable across the knowledge base, conversation feed, and test console.
- **🗂 Knowledge base is fully manageable** — built-in libraries are read-only; user-uploaded libraries can be **renamed, re-described, disabled (whole-library or per-document)** and carry a "last operation" timestamp; FAQ entries can be **disabled / enabled / deleted** (deletion goes through an in-page confirmation).
- **🧪 Bad cases drill down** — fallback / low-recall / suspected-misjudgment / **worker call failures** on the dashboard each jump to **the exact real conversation** in the effect-test feed and highlight it — not a rough approximation.
- **📦 Single file, zero dependencies, zero backend** — one `.html`, double-click to open, all data hard-coded, works offline.

## Screenshots

<!-- Drop images into docs/screenshots/ matching the filenames below; delete any line you don't have -->

![Knowledge base](docs/screenshots/01-knowledge-base.png)

![Effect test · conversation feed & trace drawer](docs/screenshots/02-effect-test.png)

![Quality dashboard](docs/screenshots/04-dashboard.png)

![Library detail · document list](docs/screenshots/05-knowledge-base-docs.png)

![Upload document wizard](docs/screenshots/06-knowledge-base-upload.png)

*Business units, brands, groups, and Q&A in the screenshots are all simulated data.*

## The eight pages

| Module | Page | Highlights |
| --- | --- | --- |
| Knowledge base | Library list | Per business unit; cards show quality score, source, status |
| | Library detail | Document list + basic info; the ops-manual library uses **real document names across four modules** |
| | Document detail | Chunks + quality bars + a right-side "single-library test"; user-library docs can be disabled |
| | FAQ library | Q-A list, each entry disable / enable / delete, with an import entry |
| | New library / upload wizard | New = name + routing note only (no file); upload runs a 4-step wizard |
| Effect test | Conversation feed | One reverse-chronological stream, filterable by group-id, three states + inline slot-filling + inline worker expansion |
| | Full-pipeline test console | Tests the whole agent pipeline; tech lead sees the trace, business lead sees assembled context |
| Quality dashboard | (tech lead only) | Hit rate / fallback rate / FCR / intent accuracy + bad-case drill-down |

## Two roles

Toggle in the top bar; the UI differentiates:

| Role | Accessible pages | Full trace | Worker assembled context | Quality dashboard |
| --- | --- | --- | --- | --- |
| **Business lead** | Knowledge base · Effect test | ❌ | ✅ | ❌ (not even shown in nav) |
| **Tech lead** | Everything | ✅ (right-side drawer, per-stage latency) | ✅ | ✅ |

> The full trace is a right-side drawer: task split → ReAct rounds → MCP calls → slot-filling → synthesis, with per-stage latency; worker timeouts / error-code paths can also be opened to see how they fall through to the fallback line.

## Quick start

No dependencies, no build. Pick one:

```bash
# 1. Just double-click 商务助理AI智能问答平台_前端原型.html

# 2. Or serve it locally
npx serve .
# then open http://localhost:3000

# 3. Or with Python
python3 -m http.server 8080
# then open http://localhost:8080
```

Suggested walkthrough: top-right, switch to **Tech lead** → go to **Effect test** → expand the first conversation's "RAG · ops-manual" to see full chunks, click chunk `#7` to jump into the document → hit "← Back" to return to the effect test → then go to the **Quality dashboard** and click a "Worker exception" case to verify the drill-down jump.

## Repo layout

```
├── 商务助理AI智能问答平台_前端原型.html   # everything: HTML + CSS + vanilla JS + mock data, single file
├── docs/screenshots/   # screenshots referenced in the READMEs
├── README.md           # English
└── README.zh-CN.md     # Chinese
```

## A few deliberate choices

- **Synthesis lives in the orchestrator, not the workers** — workers return only distilled, contracted results (RAG returns reranked chunks, business workers return clean fields); the orchestrator does the final composition. Workers are the hands that fetch; the orchestrator is the mouth that speaks.
- **Slot-filling has a single exit** — a worker missing a param doesn't ask on its own; it returns to the orchestrator, which asks the group once, so multiple workers don't talk over each other. `context_scope` (BU / brand) is auto-injected (never asked); only genuinely missing business params (e.g. an order number) trigger slot-filling.
- **No hit? Honest fallback** — empty retrieval / everything below threshold returns a fixed line ("Sorry, no relevant info found — please contact your account manager") so a human can take over. It does not make things up.
- **Built-in vs. user libraries differ** — the built-in ops-manual library is read-only and cannot be disabled; only libraries you uploaded yourself get rename / disable / delete.
- **Dangerous actions confirm in-page** — deleting an FAQ pops a custom dialog ("this cannot be undone"), not the browser's native prompt.

## Status

Portfolio-grade frontend prototype: the interactions are built out to product shape (role differences, three-axis isolation, orchestration visualization, trace drawer, bad-case drill-down), all data is simulated, and it's meant for aligning the spec and architecture, reviews, and demos. The backend is separate.

## License

MIT — see [LICENSE](./LICENSE).

## Author

**Danyang**

---

*The question this prototype is trying to answer: when a QA bot goes from "retrieve a paragraph" to "a swarm of workers coordinated by an orchestrator" — how should all that complexity be laid out in front of a person, so the people using it can understand it, trust it, and trace it.*
