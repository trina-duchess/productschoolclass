# Context Engineering & Memory: Cortex PM Chief-of-Staff Agent

> Module 4 · Context Engineering & Memory
>
> ✅ **What this validates:** the agent reasons on the right, safe inputs, by the end you'll have proven a context budget, per-source retrieve-vs-long-context decisions, and a memory map with risk mitigations.
>
> 🗂️ **How the lab maps to this file:** In **Part A** (before the lecture) you don't edit this file, you rough-draft on scratch, focused on the per-source calls in **section 2** plus a quick remember/forget + "how it rots" sketch. In **Part B** (after the lecture) you complete **all five sections**; the Lab Guide's guided builder writes this file for you to copy in and commit.

## 1. Context budget

Priority order per loop iteration, highest first:

1. **Agent line constraints** (no publish tool, propose-only, a human owns escalation) — the M1 boundaries; never dropped under budget pressure, and they win if any other input conflicts, even the task brief.
2. **`get_task`** (long-context, whole) — defines the job; everything else exists to serve this.
3. **`get_norms`** (retrieved, relevant section) — governs what's shareable/escalation *before* drafting starts.
4. **`get_activity`** (retrieved, this project, freshest) — the evidence every claim must cite; can't be dropped.
5. **`search_past_updates`** (retrieved, this project, most recent) — carries the prior metric (e.g. 41%) and format/tone precedent.
6. **`get_roadmap`** (retrieved, non-confidential slice only) — most droppable; wasn't even pulled in the happy path.

## 2. Retrieve vs. long-context: per source

For each data source, decide: **retrieve** (narrow a large/changing corpus to the relevant slice) or **long-context** (just include a bounded set you can reason over).

| Source | Size / volatility | Decision | Why (deciding factor) |
|---|---|---|---|
| `get_task` | bounded, static per run | Long-context | Size — one small static doc defining this run's whole job; nothing to search or filter, reason over all of it at once |
| `get_activity` | large, changes multiple times/day | Retrieve | Volatility — needs the freshest slice for this project only, and every PR/metric cited has to trace back to it |
| `search_past_updates` | unbounded, grows every week across every project | Retrieve | Size — only this project's most recent precedent is needed, not the whole history |
| `get_roadmap` | small but carries CONFIDENTIAL flags | Retrieve | Citation/audit — safest design is that confidential items never reach the desk at all; pull only the non-confidential slice for this project (cache it, see §3) |
| `get_norms` | small today, but must stay current | Retrieve *(flipped from Part A gut call of long-context)* | Volatility — if a rule changes (shareable info, escalation), the next run must use the new version, not a stale copy; as the playbook grows, pull only the applicable section and cite the exact rule relied on |

## 3. Retrieval quality plan

For each retrieved source, the agentic move(s) its specific failure mode demands, not all five everywhere:

| Source | Routing | Document grading | Reranking | Self-verification | Caching |
|---|---|---|---|---|---|
| `get_activity` | — | — | — | ✅ every cited PR/metric must trace to what was returned; else "can't verify" + escalate | ❌ explicitly not — changes multiple times/day, a cached pull would be stale |
| `search_past_updates` | — | ✅ drop precedent not about this project (naive keyword match has no relevance check) | ✅ most recent update first, so the "up from" metric is right | — | — |
| `get_roadmap` | — | ✅ filter out CONFIDENTIAL items before anything reaches the desk (load-bearing) | — | — | ✅ cache the filtered pull; expire weekly (each Friday run) or on roadmap file change |
| `get_norms` | ✅ route to status-update rules vs. escalation rules depending on the decision being made | — | — | — | ❌ explicitly not — volatility is why it's retrieved; caching would reintroduce staleness |

## 4. Memory map (your PM brain)

Note: per `01-agent-line/agent-line-map.md`, the current build has **zero persistence** across runs — this map is the deliberate design for what should be added, tied back to the M1 agent line (writes stay below the line: recording facts, not committing actions).

| Memory type | What Cortex stores | Scope / TTL |
|---|---|---|
| **Working** (this run) | Task brief + retrieved slices (activity, non-confidential roadmap, routed norms section, graded/reranked past-update precedent) + draft-in-progress | This run only; discarded when the run ends |
| **Episodic** (past runs) | Only **human-approved** updates, plus escalations and why, per project (not rejected drafts — a rejected draft can carry unverified numbers, e.g. an unconfirmed 41%, that would wrongly become next week's "prior"). This is also the flagged-risk history from M2's loop-spec | 90 days, then purge |
| **Semantic** (durable facts/prefs) | Distilled preferences not already in the norms/roadmap files (e.g. how leadership reads Green/Yellow/Red). Cortex may *propose* a write; only a human approves it | Durable, reviewed quarterly, never auto-trusted forever |
| **Shared** (across agents) | Realistically just the critic's verdict handed back to the drafting loop — this build is single-agent + critic (M3), not multi-agent | This run only |

**Guardrail:** confidential roadmap items must never enter semantic or episodic memory as a "learned fact" — that would let something graded out of *this* run's context quietly resurface, ungated, in a future run.

## 5. Memory risks & mitigations

| Risk | Where it bites Cortex | Mitigation |
|---|---|---|
| **Drift** | Semantic "learned preferences" quietly shift from what leadership actually wants, since they're distilled from precedent rather than restated each time | Quarterly human review of anything in semantic memory before it's trusted again; store exact figures ("43%"), never summarized ones ("most users"), so drift can't hide inside a vague paraphrase |
| **Poisoning** | A rejected/unverified draft (or a bad past-update) gets stored as if it were ground truth, then cited as fact next week | Only human-approved updates enter episodic memory; every claim in a new draft must still self-verify against `get_activity`, not against memory |
| **Staleness** | Cached roadmap slice or a semantic "fact" outlives a real change (e.g. a reversed decision); Cortex also has no notion of "today," which is why the critic flagged a valid date (`2026-07-03`) as "in the future" in the Step 0 run | Roadmap cache TTL (weekly/on-file-change, from §3); periodic review for semantic memory; pass today's date explicitly into context every run so date reasoning isn't guesswork |
| **PII / retention** | Episodic memory storing names/specifics tied to individuals (e.g. who's behind on a task) beyond when it's needed | Cap episodic retention at 90 days; store project-level facts, not individual-attributed ones, unless norms explicitly require it |
