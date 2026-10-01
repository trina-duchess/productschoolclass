# Production & Autonomy: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 5, how you'd ship it, govern it, and widen trust over time
>
> ✅ **What this validates:** you can ship it, govern it, and widen trust deliberately, by the end you'll have proven an autonomy dial, a Trust Ladder rung with its eval gate, and a governance plan.

## Autonomy Dial by segment

_Autonomy is a product decision per user, not one global setting._

| Segment | Desired autonomy | Why |
|---|---|---|
| **Team-level PM / ops lead** (routine weekly status updates) | Bounded-autonomous | Routine updates are internal and low-stakes, and this team has watched Cortex draft them accurately over time. Drafts arrive ready to review; Cortex pauses only when a draft includes a commitment, a date, or a Red/Yellow status. |
| **New eng lead** (first time using Cortex for backlog prep) | Supervised | They don't yet know where Cortex makes mistakes, so every draft and story batch waits for their approval until they've built that judgment. |
| **Leadership / exec stakeholders** (receive the final status) | Assisted | A wrong claim in front of leadership has the highest blast radius (see the false "no Sev-1" claim the critic caught in M3). Cortex drafts; a PM always reviews, edits, and sends. |

The dial changes how many below-the-line actions pause for each segment; it never moves the agent line. Posting and choosing what to escalate stay with a human for everyone.

## Trust Ladder

- **Current rung:** Assisted. Cortex drafts updates and story batches; a human reviews and acts. I want final approval before anything reaches a broader audience, and Supervised is the rung that earns that. Cortex can't yet execute an approved action (the JIT write token is spec only), and it still fails two blocking evals (EV-5, EV-1).
- **Eval gate to reach Supervised:** over **4 consecutive weekly runs**, plus every CI replay (R-1 to R-5):
  - **100% pass** on blocking evals EV-1 (0 out-of-scope reads), EV-3 (stop and escalate on missing data, 0 invented numbers), EV-5 (injection flagged in revision 1), and EV-6 (bound halts the loop)
  - EV-4 (complete, nothing released) on **≥90%** of weekly runs; EV-2 (≤6 steps) on **≥80%**
  - **Prerequisite:** JIT single-use write token built and tested
  - **Clean window = 0 trust incidents:** no unmatched number or commitment language reaching the queue unflagged, no out-of-scope read, no kill-switch pull. Critic catches are logged as near-misses, not incidents.
- **Incident record so far:** 0 trust incidents; nothing has been posted, leaked, or committed in any run. 3 logged near-misses, all caught by layers outside Cortex: the false "no Sev-1" claim (M3, critic), the P-HALO hallucination (M4, critic), and the jailbreak, missed by Cortex and caught by the critic + iteration bound (M5).

## Deployment plan

- **Runtime:** Serverless scheduled function. Cortex runs weekly (M2: Friday-noon cron, webhook backup), so pay-per-run fits better than an always-on service. Flagged-risk history persists in a small database, not in the process.
- **Operator / on-call owner:** Trina (owning PM), with the on-call platform engineer as backup. Escalation: Trina → on-call platform engineer → product lead. Runbook: pause by disabling the cron and webhook triggers; replay a failed run from its recorded trace (M5 replay set).
- **Rollback:** In order: (1) revert to the last prompt/model version that passed CI; (2) disable the misbehaving tool (e.g. `propose_stories`); (3) drop the affected segment one rung on the dial. Last resort: the M5 kill switch (disables triggers, revokes read credentials, discards unapproved queue items).
- **Monitoring:** One weekly dashboard with eval pass %, escalation rate, cost-to-serve, and trust incidents. The owning PM is alerted immediately on any bound trip, or on any unverified-source flag on a draft headed to leadership.

## ROI metrics (beyond adoption & tokens)

| Metric | Target | How it's captured |
|---|---|---|
| **Outcome: task completion** | ≥80% of weekly status updates approved with only light edits (no rewrite) | HITL queue logs approve / edit / reject, plus edit size |
| **Outcome: time saved** | ~2 hrs/week per PM vs. writing the update by hand | Baseline: time a manual update for 2 weeks; compare with time-in-queue review |
| **Cost-to-serve** | < $1 per approved update, fully loaded: model calls + retries (~$0.005/run today) + human review time | Run cost from the loop's cost counter + review minutes from queue timestamps |
| **Trust incidents** | 0 per quarter; any one incident drops the affected segment a rung | Incident log, as defined in the Trust Ladder gate |

**ROI in one sentence:** "Cortex drafted N% of weekly updates that shipped with light edits, saving ~2 hrs/PM/week at under $1 per update, with 0 trust incidents this quarter."

## Widen-autonomy decision rule

A segment moves up one rung only after it clears that rung's eval gate (the Trust Ladder thresholds) over 4 consecutive weekly runs with 0 trust incidents, and the owning PM signs off. Any trust incident drops it back a rung and restarts the 4-week clock.

## Governance & forward strategy

- **Compliance:** Customer PII, credentials, and pricing/rate data never enter a prompt. Confidential roadmap items (e.g. Orbit) may be read for context but are never written to memory, and any output matching the confidential list blocks release. Only human-approved updates are stored in memory (90-day TTL, exact figures, never summaries).
- **Safety:** Posting or approving a company-wide update, choosing what to escalate, and flagging at-risk stay above the agent line for every segment, whatever the dial setting. Cortex has no posting tool. Kill switch: one action by the owning PM (on-call engineer as backup) that disables the triggers, revokes read credentials, and discards unapproved queue items.
- **Reliability:** Max 8 iterations, revision cap 3, $0.05/run, $2/day hard cap with no auto-reset, 90s timeout. After 3 failed attempts at a required data source, Cortex stops and escalates "missing data" (EV-3). If the model is down: skip the run and notify the owning PM to write that week's update by hand. No silent fallback model, because a substitute model hasn't passed the evals.
- **Strategy:** Next, move the team-level segment from Assisted to Supervised for routine Green updates, gated by the Trust Ladder gate. First fix: EV-5 (injection flagged in revision 1), the eval Cortex fails most clearly today.
