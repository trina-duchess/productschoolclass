# Agent Line Map: Cortex PM Chief-of-Staff Agent

> Module 1 · The Agent Line
>
> ✅ **What this validates:** every risky action has a clear owner, by the end you'll have proven an above/below-the-line map with HITL checkpoints, scored on reversibility, blast radius, and measurability.

## The workflow, decision by decision

List every discrete decision or action in your agent's workflow, then score each one and place it **above** the line (a human owns it) or **below** (the agent owns it). Borderline calls get an HITL checkpoint.

| Decision / action | Reversibility (H/M/L) | Blast radius (H/M/L) | Measurability (H/M/L) | Above / Below | HITL? |
|---|---|---|---|---|---|
| Pull project state + activity | H | L | H | Below | · |
| Decide relevant context | H | L | L | Above | · |
| Draft the update | H | L | H | Below | · |
| Decide tone/commitment level | H | M | L | Below | required |
| Flag at-risk/escalation | M | H | L | Above | · |
| Choose what to escalate | M | H | L | Above | · |
| Propose a story batch (capped) | H | L | H | Below | · |
| Post an update / approve a company-wide one | L | H | M | Above | · |

## Agent anatomy (sketch)

- **Model:** `gpt-4o-mini` for routine runs; escalate to a stronger frontier model when an issue is flagged for escalation, to help with review and generating mitigation options.
- **Tools:** project + activity lookup, past-update search, roadmap, team norms lookup, story proposal (capped) — no publish/post tool exists, by design, so nothing can ship without a human.
- **Memory:** nothing persists across runs — roadmap, norms, and past updates are read fresh from fixtures each time; Cortex doesn't remember prior flags or decisions on its own.
- **Loop:** _placeholder, defined in M2 loop-spec.md_
- **Bounds:** _placeholder, defined in M5 bounds-and-evals.md_
- **Evals:** _placeholder, defined in M5 bounds-and-evals.md_

## The golden rule, applied

1. **Pull project state + activity** — sits below the line because it's easy to reverse, has a low blast radius, and is easy to verify. Deciding factor: blast radius.
2. **Decide relevant context** — sits above the line because it's easy to reverse and has a low blast radius, but is hard to verify. Deciding factor: measurability.
3. **Draft the update** — sits below the line because it's easy to reverse, has a low blast radius, and is easy to verify. Deciding factor: reversibility.
4. **Decide tone/commitment level** — sits below the line with a required human check because it's easy to reverse and has a moderate blast radius, but is hard to verify. Deciding factor: measurability.
5. **Flag at-risk/escalation** — sits above the line because it's moderately reversible and has a high blast radius, and is hard to verify. Deciding factor: blast radius.
6. **Choose what to escalate** — sits above the line because it's moderately reversible and has a high blast radius, and is hard to verify. Deciding factor: blast radius.
7. **Propose a story batch (capped)** — sits below the line because it's easy to reverse, has a low blast radius, and is easy to verify. Deciding factor: measurability.
8. **Post an update / approve a company-wide one** — sits above the line because it's hard to reverse and has a high blast radius, though moderately verifiable. Deciding factor: reversibility.

## Hardest call

**Choose what to escalate** was the hardest call — it went back and forth before landing above the line. Blast radius was the deciding factor: what reaches leadership carries real weight, and only risks paired with mitigation plans should make that cut, which takes human judgment Cortex can't own.
