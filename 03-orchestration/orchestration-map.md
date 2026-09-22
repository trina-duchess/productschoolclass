# Orchestration Map: Cortex PM Chief-of-Staff Agent

> Module 3 · Orchestration & Subagents, ★ Deliverable 3
>
> ✅ **What this validates:** nothing advances unchecked, by the end you'll have proven a justified topology, a roster, and a validator with a defined fail action.
>
> Builds on your M2 Loop Spec. Only split one agent into a team when there's a real reason, coordination has a cost.

## 1. Why split? (or why not)

Cortex needs an independent validator (critic) to evaluate and correct the results. Do not need both Separation of Concern and Validator. Just validator.

## 2. Topology

**Pattern:** _single+subagents · sequential · parallel+aggregate · hierarchical_

```
[Inbound PM task] → [Cortex: pulls data, drafts update + stories]
                  → [Validator], fail → back to Cortex (max 3 revisions) → escalate, pass → [PM review checkpoint] → queued
```

## 3. Roster

| Agent | Responsibility | Loop Spec |
|---|---|---|
| Cortex | Pulls project data, drafts status update + story proposals | Cron (primary) + hook (backup), same trigger for both |
| Critic | Independent validator — checks Cortex's draft against house norms, traceability, queue cap; called inline per draft | Same loop as Cortex, separate call/context |

## 4. Communication & hand-offs

Cortex passes its proposed draft output plus the source data it pulled to critic.review() as a direct in-process function call — no MCP/A2A, no separate service. The critic passes back a JSON verdict ({"verdict": "pass"|"fail", "reasons": [...]}), which Cortex uses to either advance the draft or revise/escalate.

## 5. The validator

Checks:
1. Update references the correct project + PR/issue IDs
2. Every figure is traceable to pulled data (no invented numbers)
3. Story batch stays within the queue cap (or flags if it exceeds it)
4. Tone matches house style; no commitments Cortex may not make (dates/deadlines, leadership commitments)

Fail-action: Revise — return the draft to Cortex with the failure noted, rerun up to the revision cap.

Revision cap: 3. If still failing after 3 revisions, stop (don't force a draft through) — matches the loop-spec's stuck/give-up condition.

Pass-action: A passing draft advances to the PM review checkpoint. It does not auto-send or auto-post.

## 6. State: shared vs isolated

Shared: the source data pulled from fixtures, and the draft itself. Also shared: the critic's verdict and reasons — Cortex needs these to revise, since agent.py appends the critic's rejection reasons as a user message before looping back for another attempt.

Isolated: the critic's own system prompt/context (CRITIC_SYSTEM). It never sees Cortex's drafting instructions (CORTEX_SYSTEM) or Cortex's process — that separation is the whole point.

## 7. Cost & latency budget

The critic adds one model call per draft attempt (one extra call per revision). Worst case at the cap of 3: three extra critic calls, observed run cost ~$0.0045 total. This adds the latency of 3 critic round-trips before a draft reaches the PM, instead of reaching them after just one.
