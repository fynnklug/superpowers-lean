# Scoped Re-Review Prompt Template

Dispatch the `superpowers:sdd-reviewer` agent in re-review mode after a fix
round. This is not a fresh review — the full review already happened. The
re-reviewer verdicts each prior finding and checks the fix diff for new
breakage.

Do not pass a `model`: scoped re-reviews of small fix diffs belong on the
agent's default tier.

```
Subagent (superpowers:sdd-reviewer):
  description: "Re-review Task N fix round R"
  prompt: |
    Scoped fix re-review. Verdict each finding below, then inspect the fix
    diff — nothing else.

    **The task:** [BRIEF_FILE]
    **The implementer's report** (fix reports are appended at the end): [REPORT_FILE]
    **Fix diff:** [DIFF_FILE] (fix base [FIX_BASE_SHA] → head [HEAD_SHA])

    ## Findings Under Verification

    [FINDINGS]

    ## Scope

    Verdict every finding. Inspect the fix diff for problems the fix itself
    introduced. Do NOT re-review code the fix did not touch: an issue entirely
    outside the fix diff goes under Out-of-Scope Observations — it does not
    block this task and does not extend the loop. A broad whole-branch review
    happens after all tasks are complete.

    The implementer re-ran the tests covering the amended code and appended the
    results to the report file. Confirm the fix report names the covering
    tests and shows their output, and verify its claims against the diff.

    ## Output Format

    ### Finding Verdicts
    For each finding above, in order:
    - **[finding one-liner]** — ADDRESSED | NOT ADDRESSED, with file:line
      evidence. "Attempted" is not addressed: the specific defect must no
      longer exist.

    ### New Breakage in the Fix Diff
    What the fix broke or introduced, with severity and file:line. "None" if clean.

    ### Out-of-Scope Observations
    Issues entirely outside the fix diff. Non-blocking. "None" if none.

    ### Verdict
    **Fix round:** [All findings addressed, no new Critical/Important breakage |
    Findings remain open] — list the open ones.
```

**Placeholders:**
- `[BRIEF_FILE]` — the same brief the implementer worked from
- `[REPORT_FILE]` — the implementer's report file (fix reports appended)
- `[DIFF_FILE]` — the path `scripts/review-package PLAN_FILE FIX_BASE HEAD` printed
- `[FIX_BASE_SHA]` — the head the previous review saw
- `[HEAD_SHA]` — current commit
- `[FINDINGS]` — the Critical/Important findings and confirmed spec gaps from
  the previous review, copied verbatim, one per bullet

**Re-reviewer returns:** per-finding verdicts (ADDRESSED / NOT ADDRESSED), new
breakage in the fix diff, out-of-scope observations, and a round verdict.
