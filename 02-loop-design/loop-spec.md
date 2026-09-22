# Loop Spec: Cortex PM Chief-of-Staff Agent

> Module 2 · Loop Engineering, ★ Deliverable 2
>
> ✅ **What this validates:** the agent knows when to run and when to stop, by the end you'll have proven a one-page Loop Spec with a trigger, a definition of "done," and explicit stop conditions.
>
> Your one-page blueprint for how the work you handed to the agent (M1) actually *runs*.
> An agent is just a prompt that fires itself, this spec says when it fires, what "done" means, and what it needs to do the job. Living document; refine as the course progresses.

## 1. Trigger & loop type

**Chosen type:** Cron (primary) + Hook (backup)

**Why this type:** Cortex needs to collect project data from team tools and complete a draft for human review by noon each Friday — a fixed weekly deadline drives the primary cron trigger. A backup hook also fires if a critical update lands after the cron run, so an urgent item doesn't sit unseen until next week's cycle.

**Ruled out:**
- *Heartbeat* — overkill for status updates; nothing here needs continuous polling.
- *Goal* — the whole run is anchored to a fixed weekly deadline (data collected and drafted by noon Friday), not an open-ended loop waiting on validation.

**Idempotency / dedupe:** Cortex tracks each processed message by ID and timestamp, so the same inbound event firing the hook twice doesn't produce two drafts.

## 2. Goal / definition of done

Project data has been pulled from team tools, a status draft has been written and passed the critic's review, and any proposed backlog stories are queued — all held for human review. Cortex never posts or sends anything itself.

## 3. Stop conditions

| Condition | What it looks like | What happens |
|---|---|---|
| **Success** | Draft passes the critic's validation and is queued at the HITL checkpoint by noon Friday | Held for review, nothing posted |
| **Stuck / give up** | A data source can't be reached after 3 attempts, or the critic rejects the draft without a pass across 3 revisions (`MAX_REVISIONS`) or 8 total loop iterations (`MAX_ITERATIONS`), whichever comes first | Stop, log the reason, don't force a draft through |
| **Escalate to human** | Content touches an already above-the-line item: a flagged at-risk project, anything implying a leadership commitment or deadline, or a story batch over cap | Immediate HITL flag, separate from the normal weekly queue |

## 4. State

Cortex now persists a per-project record of risks it has already flagged, so it doesn't re-raise the same risk every Friday. Everything else (roadmap, team norms, past updates) still reads fresh from source each run — only the flag history carries over. Scope: risk-flag history stays scoped to its own project, no cross-project leakage.

## 5. The five things a loop can lean on

| Component | For Cortex |
|---|---|
| **Work tree** (isolated workspace per run, a git worktree) | Not needed — Cortex drafts text and never edits code or files in the repo, so there's nothing to isolate per run. |
| **Skills** (reusable capabilities) | Not formalized yet — the current tool functions in `tools.py` already serve this role informally; whether they need a distinct skills layer is open for a later module. |
| **Plugins / connectors** (tools & access, optional if you don't have one yet) | Not wired yet — `tools.py` currently reads local fixture files, not a live Jira key or GitHub/Slack API. Plan: wire real connectors once this design passes review. |
| **Subagents** (independent check when the loop can't grade itself) | Already built — `critic.py` is a working independent check on the draft today, not a placeholder. |
| **State tracking** | Persists a per-project record of risks already flagged, so the same risk isn't re-raised every cycle; scoped per project. |

> Context plan (M4) and the hand-off to bounds & evals (M5) come in later modules, you'll add them to their own deliverables then, not here.

## Link to live loop

_[path to your agent in `00-build/`]_
