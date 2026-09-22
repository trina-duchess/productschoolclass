# Prototype: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 1, the working agent demo
>
> ✅ **What this validates:** the agent actually runs end to end, by the end you'll have proven it with real screenshots of your Cortex across the six required moments (M2 to M6).

## What it does

_One paragraph: the agent in action, end to end._

## How you built it

- **Coding agent:** _which one you directed (Claude Code / Cursor / Codex)_
- **Model + bounds:** _model used, max iterations, cost cap, queue cap_
- **Repo / config:** _path to your build in `00-build/`_
- **Live link:** _[shareable URL, optional bonus]_

## Screenshots (required, collected M2 to M6)

Real screenshots of *your* Cortex running. These are the `00-build/CORTEX-ANATOMY.md` set and they are required, a link alone is not enough.

| # | Screenshot | What it shows | From |
|---|---|---|---|
| 1 | _[img]_ | happy-path run: a real drafted update + the HITL checkpoint (queued, not posted) | M2 |
| 2 | _[img]_ | the critic rejecting a bad draft (revise/block) | M3 |
| 3 | _[img]_ | a grounded update citing pulled activity + a caught hallucination | M4 |
| 4 | _[img]_ | jailbreak refused + escalated | M5 |
| 5 | _[img]_ | an iteration/cost/queue bound halting a runaway | M5 |
| 6 | _[img]_ | end-to-end run | M6 |

## How to run it

_Minimal steps for someone to reproduce the demo (env vars, and the command or the coding-agent prompt you used)._

## Critic rejection evidence (M3 required capture)

Critic rejected Cortex's status-update draft on all 3 revision attempts before escalating to a human, nothing posted:

- Verdict 2 reasons: language could imply a confident status on a CONFIDENTIAL roadmap item, without clearly separating stated facts from inference
- Verdict 3 reasons: claims not clearly traceable to pulled data (a "no Sev-1 issues open" claim unconfirmed), and queued/unapproved stories presented in a way that could read as finalized

Final: escalated after hitting the iteration cap (MAX_ITERATIONS=8) at essentially the same moment as the 3rd revision rejection — both limits triggered together. Draft held for human review at run-output\status-update-happy.md; nothing posted.

```
================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "fail",
  "reasons": [
    "The text generated claims the status is 'Green - On Track', but does not explicitly tie this status to the results of the roadmap or previous status updates, which could lead to confusion about the evaluation criteria.",
    "The proposed output fails to state that the reviewed stories have been queued for approval, making it look like they are already accepted or finalized.",
    "The proposed output implies a detailed next step list, which could be interpreted as a commitment to specific actions rather than a summary of intent. It should clarify that these are planned actions and that they are related to the stories proposed from PRD-Northstar-v3.",
    "The output states 'Current Flags: No Sev-1 issues open,' but it does not provide sufficient context for what this means in relation to the project objectives or actual current issues if any arise.",
    "The summary mentions a metric increase without contextualizing the previous week's metric clearly as being part of the weekly comparisons, slightly leading to an impression of 'invented progress'. The metrics should match clearly against previously reported data."
  ]
}

-> critic rejected; revision 1/3

================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "fail",
  "reasons": [
    "The proposed output references the project P-NORTH (Northstar) and real activities (merging of PRs and opening of issues) correctly from the pulled data.",
    "However, the check for disclosing only shareable information reveals that the phrases like 'improving the time-to-first-value in self-serve onboarding' and 'aligns with our roadmap goal of lifting the activation rate' could be construed as referencing the roadmap, and while sharing project status, wording could unintentionally imply a confident status on a CONFIDENTIAL roadmap item.",
    "Furthermore, there's no clear separation indicating that the details about the roadmap objectives shared are based solely on the information from the valid public updates or metrics without assuming further details that might not have been explicitly stated in the output."
  ]
}

-> critic rejected; revision 2/3

================================================================
CRITIC, independent validation
================================================================
{
  "verdict": "fail",
  "reasons": [
    "The output discusses project status, activities, and metrics that are not directly validated against the pulled data in a clear manner, specifically the statement on 'current flags' that indicates no Sev-1 issues open; the project's data does not provide this confirmation.",
    "The 'Next Steps' section refers to proposed stories but does not clarify that they were queued and are awaiting approval, potentially misleading the recipient into believing they are finalized.",
    "The output does not acknowledge that all actions and stories proposed are still in a queued status, even while stating that the actions will be pursued following approval, which could imply a higher certainty about execution than is warranted."
  ]
}

-> critic rejected; revision 3/3

================================================================
MAX ITERATIONS (8) reached without finishing. Escalating. Run cost ~ $0.0045
================================================================

================================================================
LAST DRAFT (held, NOT posted, escalated to a human)
================================================================
**Revised Status Update for Project Northstar (P-NORTH)**

**Current Status:** Green - On Track
This status is based on recent project activities without disclosing any confidential information. We have successfully merged two critical PRs, contributing to the ongoing development of features aimed at enhancing onboarding experiences.

**Recent Activity:**
- **Merged PRs:**
  - **#812**: New activation checklist UI (Date: 2026-06-29)
  - **#815**: Instrument step-completion events (Date: 2026-06-30)
- **Open Issues:**
  - **#818**: Empty-state copy needs review (Severity: normal)
- **Metric:** The activation rate has increased from 39% to 41% week-over-week, providing clear evidence of improved performance relative to the previous period.

**Next Steps:**
We plan to focus on the following actions as part of our continuous improvement strategy. These actions correspond to stories proposed from PRD-Northstar-v3 and are intended to enhance the overall onboarding process:
- Implementing empty-state guidance
- Designing contextual tips
- Setting up a day-2 milestone email
These actions will be pursued following the pending approval of the proposed stories.

**Risks and Blockers:**
- **Current Flags:** There are currently no Sev-1 issues open, indicating a stable development environment. This favorable condition allows the team to concentrate on upcoming initiatives without significant interruptions.

**Proposed Stories Status:**
The following top stories are queued for your review ahead of our sprint planning:
- Implement empty-state guidance
- Design contextual tips
- Setup day-2 milestone email

These stories are pending your approval and should not be interpreted as finalized.

---

DONE: Status update revised to ensure clarity around shareable information and stories queued for your review.

Why it was held: max iterations (8) reached

Saved draft -> run-output\status-update-happy.md  (for your review, nothing was posted)
```
