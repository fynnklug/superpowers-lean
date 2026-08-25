# Task Reviewer Prompt Template

Dispatch the `superpowers:sdd-reviewer` agent. Its standing contract — review
method, don't-trust-the-report, test policy, calibration, evidence rules —
lives in the agent definition. This prompt carries the task's inputs and the
output format.

The reviewer reads the task's diff once and returns two verdicts: spec
compliance and code quality.

Do not pass a `model` — the agent's tier is set for task-scoped diffs. Override
to `opus` only for a diff whose risk genuinely warrants it (a subtle
concurrency or security change), and say why in the ledger.

```
Subagent (superpowers:sdd-reviewer):
  description: "Review Task N (spec + quality)"
  prompt: |
    Full task review. Two verdicts: spec compliance, then code quality.

    **What was requested:** [BRIEF_FILE]
    **What the implementer claims:** [REPORT_FILE]
    **Diff under review:** [DIFF_FILE] (base [BASE_SHA] → head [HEAD_SHA])

    Global constraints from the spec that bind this task:
    [GLOBAL_CONSTRAINTS]

    ## Part 1: Spec Compliance

    Compare the diff against what was requested:

    - **Missing** — requirements skipped, or claimed without being implemented
    - **Extra** — features not requested, over-engineering, unneeded nice-to-haves
    - **Misunderstood** — right feature built the wrong way, wrong problem solved

    If the brief lists several files each with its own change (a batched
    dispatch), check the diff file by file: every listed file must have its
    hunk. A listed file the diff never touches is Missing, no matter how clean
    the rest of the batch looks.

    If a requirement cannot be verified from this diff alone (it lives in
    unchanged code or spans tasks), report it as ⚠️ rather than broadening
    your search.

    ## Part 2: Code Quality

    - **Code:** separation of concerns, error handling, DRY without premature
      abstraction, edge cases
    - **Tests:** do new and changed tests verify real behavior rather than
      mocks? are the task's edge cases covered?
    - **Structure:** one clear responsibility per file with a well-defined
      interface? units independently understandable and testable? following the
      plan's file structure? did this change create files that are already
      large, or significantly grow existing ones? (Don't flag pre-existing
      sizes — judge what this change contributed.)

    ## Output Format

    ### Spec Compliance
    ✅ Spec compliant | ❌ Issues found: [what's missing/extra/misunderstood, file:line]
    ⚠️ Cannot verify from diff: [what, and what the controller should check —
    report alongside the ✅/❌ verdict for everything you could verify]

    ### Strengths
    [Specific.]

    ### Issues
    #### Critical (Must Fix)
    #### Important (Should Fix)
    #### Minor (Nice to Have)

    ### Assessment
    **Task quality:** [Approved | Needs fixes]
    **Reasoning:** [1-2 sentences]
```

**Placeholders:**
- `[BRIEF_FILE]` — REQUIRED: the same brief the implementer worked from
  (`scripts/task-brief PLAN N` prints the path)
- `[REPORT_FILE]` — REQUIRED: the implementer's detailed report file
- `[DIFF_FILE]` — REQUIRED: the path `scripts/review-package PLAN_FILE BASE HEAD`
  printed (the package never enters the controller's context)
- `[BASE_SHA]` / `[HEAD_SHA]` — the commit before this task, and its head
- `[GLOBAL_CONSTRAINTS]` — binding requirements copied verbatim from the plan's
  Global Constraints section or the spec: exact values, formats, and stated
  relationships between components. Not process rules — those are in the agent.

**Reviewer returns:** Spec Compliance verdict (✅/❌/⚠️), Strengths, Issues
(Critical/Important/Minor), Task quality verdict.
