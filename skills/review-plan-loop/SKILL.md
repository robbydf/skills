---
name: review-plan-loop
description: Use when a plan, design doc, or spec (a regular implementation plan, or a superpowers brainstorming design / writing-plans spec) is drafted and needs an independent review-and-fix loop before the user approves it. Triggers - "review the plan", "review-plan-loop", "get the plan reviewed", "have Opus/Sonnet review the spec".
---

# Review Plan Loop

An independent reviewer subagent critiques the plan; you fix every Critical and Important finding; the same reviewer re-reviews. Repeat until clean or **5 rounds**, then ask the user to approve the plan.

## Inputs

- **Plan target:** the plan/design/spec file(s) just written in this session. If unclear, look for the most recent file under `docs/superpowers/specs/`, `docs/superpowers/plans/`, `docs/plans/`, or the plan-mode plan file. If still ambiguous, ask which file.
- **Reviewer model:** `opus` (Opus 5.5) by default. Use `sonnet` (Sonnet 5.5) only if the user asked for it (e.g. "review-plan-loop with sonnet").
- **Max rounds:** 5.

## Round 1: dispatch the reviewer

Call the Agent tool with `subagent_type: "general-purpose"`, `model: "<opus|sonnet>"`, and a prompt containing, in order:

1. Role: "You are an independent senior reviewer of a software plan. You did not write it. Be rigorous and specific."
2. The plan file path(s) and the repo root; tell it to read the plan AND the relevant code it references so findings are grounded in reality.
3. The goal/requirements the plan must satisfy (from the user's request or the spec), in 2-5 lines.
4. What to check: correctness and feasibility against the actual codebase, missing requirements, wrong assumptions about existing code/APIs, ordering/dependency errors between steps, missing tests or verification steps, risky migrations/data loss, security issues, scope creep, ambiguity that would block an implementer.
5. Output contract - return ONLY a list of findings, each with:
   - `id` (R<round>-<n>), `severity`: **Critical** (plan will fail / cause harm), **Important** (likely defect, significant gap), **Minor** (nice-to-have, style, wording)
   - location (section / step), problem, concrete suggested fix
   - Then a final line: `VERDICT: APPROVED` if there are no Critical or Important findings, otherwise `VERDICT: CHANGES_REQUIRED`.
6. "Do not edit any files."

Keep the returned agent ID/name - re-reviews go to the same reviewer.

## Fix step (every round)

For each Critical and Important finding:
- Verify it against the code (superpowers:receiving-code-review rigor: no blind agreement). If it is wrong, do not change the plan; record a short rebuttal with evidence.
- Otherwise edit the plan file to fix it.

Minor findings: apply only if trivial and clearly correct; otherwise list them for the user.

Keep a running log per round: finding id, severity, action (fixed / rebutted / deferred).

## Re-review (rounds 2..5)

Send the same reviewer a message via `SendMessage` (to its agent ID): "Round N re-review. I changed: <bullet list of fixes by finding id>. I rebutted: <id: reason>. Re-read the plan file(s) and re-review the whole plan. Confirm whether prior Critical/Important findings are resolved, and report any NEW findings, same output contract and VERDICT line."

If SendMessage is unavailable or the agent is gone, dispatch a fresh Agent (same model) with the Round 1 prompt plus the fix/rebuttal log.

## Stop conditions

- `VERDICT: APPROVED` (no open Critical/Important) -> stop.
- Round 5 completed -> stop, even if findings remain.
- The reviewer repeats a finding you already rebutted with evidence -> treat as disagreement, do not loop on it; surface it to the user.

## Finish: ask for plan approval

Report to the user:
- Rounds run (N/5) and final verdict.
- Table: finding id, severity, one-line summary, action taken.
- Any unresolved Critical/Important findings or disagreements (call these out first).
- Deferred Minor findings.

Then ask the user to approve the plan (AskUserQuestion, or the wrap-up question): options like "Approve plan" / "Request changes". Do NOT start implementing until the user approves.

## Common mistakes

- Reviewing the plan yourself instead of dispatching a subagent - the point is an independent model.
- Starting a fresh reviewer every round (loses context; reviewer re-raises settled points). Reuse via SendMessage.
- Fixing Critical/Important findings without checking them against the code.
- Exceeding 5 rounds, or silently stopping without the approval question.
