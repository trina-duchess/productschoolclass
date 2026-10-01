# Build Insights: Cortex PM Chief-of-Staff Agent

> Module 6 · ★ Deliverable 4, what you learned building it
>
> ✅ **What this validates:** you can reflect on what building it taught you, by the end you'll have proven the friction, the learning, and the aha that changes how you'd design your next agent.

## Friction

The critic. It rejected accurate drafts over and over, most memorably flagging a "no Sev-1" claim as unconfirmed, and the same happy-path task escalated at the 8-iteration cap on one run and passed cleanly on the next. Tuning a validator that's strict enough to catch a hallucination but doesn't block good work was harder than building the agent.

## Learning

1. The safety comes from layers outside the model. In the jailbreak test Cortex itself missed the injection; the critic, the iteration cap, and the missing posting tool are what kept anything from going out.
2. A limit that lives only in the spec isn't a bound. My spec scoped reads to the task's projects, but the build didn't enforce it, and Cortex read an out-of-scope project.
3. Evals are acceptance criteria. Writing pass/fail thresholds made "is it ready?" a measurable question instead of a feeling.

## Aha moment

One good demo proves nothing. The same task failed and then passed on back-to-back runs, which is exactly why autonomy has to be earned over a window of runs, not granted after one.

## What you'd do differently

Write the evals first and build to them, and enforce scope and the write token in code from day one, not leave them as spec. I'd also tune the critic against a set of known-good drafts so it stops rejecting accurate updates.
