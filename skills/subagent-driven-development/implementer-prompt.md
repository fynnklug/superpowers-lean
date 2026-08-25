# Implementer Subagent Prompt Template

Dispatch the `superpowers:sdd-implementer` agent. Its standing contract —
self-review, escalation, TDD evidence, report format, no-subagents — lives in
the agent definition. This prompt carries only what is specific to THIS task.

Do not pass a `model`: the agent runs on a cheap tier by design. Override it
to `opus` only for fix-round escalation (see SKILL.md, The fix loop).

```
Subagent (superpowers:sdd-implementer):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    Read your task brief first — it is your requirements, and its exact
    values are to be used verbatim: [BRIEF_FILE]

    Work from: [directory]
    Write your full report to: [REPORT_FILE]

    ## Context

    [One line on where this task fits in the project.]

    ## Interfaces and Decisions From Earlier Tasks

    [Signatures, names and choices the brief cannot know. Omit this section
    for Task 1.]

    ## Rulings

    [Your resolution of any ambiguity you noticed in the brief, and pointers
    to ledger entries for findings parked in the area this task touches.
    Omit if none.]

    ## Global Constraints

    [The binding requirements from the plan's Global Constraints section,
    copied verbatim.]
```

**Placeholders:**
- `[BRIEF_FILE]` — REQUIRED: `scripts/task-brief PLAN N` prints the path
- `[REPORT_FILE]` — REQUIRED: named after the brief
  (`…/task-N-brief.md` → `…/task-N-report.md`)
- `[directory]` — the worktree the task runs in

**Implementer returns:** Status (DONE / DONE_WITH_CONCERNS / BLOCKED /
NEEDS_CONTEXT), commits, a one-line test summary, concerns, and the report
file path.
