---
name: review-code-loop
description: Use when implementation work on a branch is done (or a chunk of it) and the code changes need an independent review-and-fix loop before opening a pull request. Triggers - "review the code", "review-code-loop", "get my changes reviewed", "have Sonnet/Opus review the diff before the PR".
---

# Review Code Loop

An independent reviewer subagent reviews the code changes; you fix every Critical and Important finding; the same reviewer re-reviews. Repeat until clean or **8 rounds**, then ask the user whether to open a PR and whether it should be a draft.

## Inputs

- **Review target:** the changes on the current branch. Determine the base: `git merge-base HEAD origin/<default-branch>` (default branch from `git symbolic-ref refs/remotes/origin/HEAD`, fall back to `main`/`master`). Scope = committed diff `base..HEAD` plus uncommitted changes (`git status`, `git diff`). If the user named files, a commit range, or a plan, use that instead.
- **Requirements:** the plan/spec this code implements (e.g. under `docs/superpowers/plans/` or `docs/superpowers/specs/`), if one exists, plus the user's request.
- **Reviewer model:** `sonnet` (Sonnet 5.5) by default. Use `opus` (Opus 5.5) only if the user asked for it (e.g. "review-code-loop with opus").
- **Max rounds:** 8.

## Round 1: dispatch the reviewer

Call the Agent tool with `subagent_type: "general-purpose"`, `model: "<sonnet|opus>"`, and a prompt containing, in order:

1. Role: "You are an independent senior code reviewer. You did not write this code. Be rigorous and specific."
2. Repo root, base SHA, head SHA, and the exact commands to see the changes (`git diff <base>...HEAD`, `git diff` for uncommitted work), plus the changed-file list.
3. The requirements in 2-5 lines and the plan/spec path if one exists.
4. What to check: correctness bugs and edge cases, requirement/plan conformance, error handling, security (injection, authz, secrets), data loss / migration safety, concurrency, test coverage of the new behavior, breaking API changes, performance traps, consistency with the surrounding codebase's conventions. Tell it to read surrounding code, not just the diff, and it may run the tests/linters but must not edit files.
5. Output contract - return ONLY a list of findings, each with:
   - `id` (C<round>-<n>), `severity`: **Critical** (bug, security hole, data loss, broken build/tests), **Important** (likely defect, missing tests for new behavior, requirement gap, significant design problem), **Minor** (style, naming, nits)
   - `file:line`, problem, concrete suggested fix
   - Then a final line: `VERDICT: APPROVED` if there are no Critical or Important findings, otherwise `VERDICT: CHANGES_REQUIRED`.

Keep the returned agent ID/name - re-reviews go to the same reviewer.

## Fix step (every round)

For each Critical and Important finding:
- Verify it (superpowers:receiving-code-review rigor: no blind agreement). If it is wrong, do not change the code; record a short rebuttal with evidence.
- Otherwise fix it. Add or update tests when the finding is a behavior bug.
- Run the project's tests/linters/build after the round's fixes. Do not send for re-review with a failing build.

Minor findings: apply only if trivial and clearly correct; otherwise list them for the user.

Do not commit unless the user has asked you to commit in this session; leave fixes in the working tree otherwise. Keep a running log per round: finding id, severity, action (fixed / rebutted / deferred).

## Re-review (rounds 2..8)

Send the same reviewer a message via `SendMessage` (to its agent ID): "Round N re-review. I changed: <bullet list of fixes by finding id, with files>. I rebutted: <id: reason>. Test status: <result>. Re-review the full current diff against <base>. Confirm whether prior Critical/Important findings are resolved, and report any NEW findings, same output contract and VERDICT line."

If SendMessage is unavailable or the agent is gone, dispatch a fresh Agent (same model) with the Round 1 prompt plus the fix/rebuttal log.

## Stop conditions

- `VERDICT: APPROVED` (no open Critical/Important) -> stop.
- Round 8 completed -> stop, even if findings remain.
- The reviewer repeats a finding you already rebutted with evidence -> treat as disagreement, do not loop on it; surface it to the user.

## Finish: ask about the PR

Report to the user:
- Rounds run (N/8), final verdict, final test/build status.
- Table: finding id, severity, one-line summary, action taken.
- Any unresolved Critical/Important findings or disagreements (call these out first).
- Deferred Minor findings.

Then ask (AskUserQuestion, or the wrap-up question):
1. "Open a pull request?" - Yes / No.
2. If yes: "Draft or ready for review?" - Draft / Ready for review.

Only on a yes: commit any remaining changes (if the user agreed to commit), push the branch, and create the PR with `gh pr create` (add `--draft` if Draft), title and body summarizing the change and the review rounds. If the user says no, stop and leave the branch as is.

## Common mistakes

- Reviewing the code yourself instead of dispatching a subagent - the point is an independent model.
- Starting a fresh reviewer every round (loses context; reviewer re-raises settled points). Reuse via SendMessage.
- Re-review requested while tests are failing.
- Exceeding 8 rounds, opening a PR without asking, or forgetting the draft question.
