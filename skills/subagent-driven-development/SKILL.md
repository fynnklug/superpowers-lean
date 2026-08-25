---
name: subagent-driven-development
description: Use when executing implementation plans with independent tasks in the current session
---

# Subagent-Driven Development

Execute a plan by dispatching a fresh implementer subagent per task, a task
review (spec compliance + code quality) after each, and a broad whole-branch
review at the end.

**Why subagents:** you delegate to agents with isolated context. They never
inherit your session's history — you construct exactly what they need. That
keeps them focused and preserves your context for coordination.

**Narration:** between tool calls, narrate at most one short line — the ledger
and the tool results carry the record.

**Continuous execution:** do not pause to check in between tasks. Execute all
tasks without stopping. "Should I continue?" prompts and progress summaries
waste your human partner's time — they asked you to execute the plan.

**Rulings, not stalls.** A running plan does not wait on a human. Conflicts,
ambiguities, plan defects, a cap you would have asked to exceed — decide them.
The spec is the binding authority, the plan is its argument, and your judgment
settles what neither answers. Record every decision in the ledger as
`Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep
going. A wrong ruling costs rework your partner can see and undo; a session
parked on a question costs their whole day and buys nothing.

Four things stop you, and only these: an irreversible or destructive
operation; a security-sensitive action; a side effect outside this worktree
that norms say you ask about first (a merge, a push to a shared branch, a
publish); and a plan so broken that every path forward is a guess.

## When to Use

You have an implementation plan whose tasks are mostly independent, and you
are staying in this session. Tightly coupled tasks or no plan → brainstorm or
execute manually. Want a parallel session instead → superpowers:executing-plans.

## The Agents

Three agent types carry this process. Their standing contracts live in their
definitions, so your dispatch prompts stay short — never re-state a contract
the agent already has.

| Role | Agent | When |
|------|-------|------|
| Implementation, fix rounds | `superpowers:sdd-implementer` | every task |
| Task review, scoped re-review | `superpowers:sdd-reviewer` | after every task |
| Whole-branch review | `superpowers:code-reviewer` | once, at the end |

Each agent's model and reasoning effort are set in its definition — do NOT
pass a `model` at dispatch. The tiers are deliberate: implementers and task
reviewers run cheap because a well-specified task is mostly transcription plus
testing, and the final review runs expensive because it is the one that must
catch what everything else missed.

Override the model only in the two cases this skill names: fix-round
escalation, and a task-review diff whose risk genuinely warrants it. Ledger
the reason when you do.

## Setup

Ensure the work happens in an isolated workspace: use
superpowers:using-git-worktrees to create one or verify the existing one.
Never start implementation on a main/master branch without explicit consent.

Conversation memory does not survive compaction. Controllers that lost their
place have re-dispatched entire completed task sequences — the single most
expensive failure observed. Track progress in a ledger file, not only in todos.

- Each plan owns a workspace: run `scripts/sdd-workspace PLAN_FILE` — it prints
  the plan's git-ignored directory (`<repo-root>/.superpowers/sdd/<plan-basename>/`),
  home to every artifact for THIS plan: ledger, briefs, reports, review
  packages. Another plan's directory is never yours to read or write.
- Check for this plan's ledger at `<workspace>/progress.md`. If its first line
  names your plan file, tasks with a `Task <N>: complete` line are DONE — do
  not re-dispatch them; resume at the first task without one. A task whose last
  line is a fix round is mid-loop: resume at the next round. A ledger naming a
  different plan file — or a stray one at the old flat path
  `.superpowers/sdd/progress.md` — belongs to another plan: leave it and start
  your own, fresh.
- Create the ledger with its identity as the first line:
  `# SDD ledger — plan: <plan file path>`.
- The ledger is your recovery map: the commits it names exist in git even when
  your context no longer remembers creating them. After compaction, trust the
  ledger and `git log` over your own recollection.
- `git clean -fdx` destroys the workspace (git-ignored scratch); recover from
  `git log`.

Read the plan once, note its context and Global Constraints, and create a todo
per task. If the plan names a Spec, read that too: the spec is the authority
the plan argues from, and conflicts inside the plan resolve against it. A plan
with no reachable spec gets a ledger note saying so — rulings made without one
are provisional.

**Pre-flight scan.** Before dispatching Task 1, scan the plan for conflicts:
tasks that contradict each other or the Global Constraints, and anything the
plan mandates that the review rubric treats as a defect (a test that asserts
nothing, verbatim duplication of a logic block).

The output is a table, not a verdict. One row for every pair of tasks sharing
a file or interface: the two tasks, what one produces against what the other
consumes, what you found. One row per task: whether its own text agrees with
itself. "The scan is clean" without those rows is not a scan you ran.

Write the table to the ledger, rule on everything it surfaces — the spec is
binding — record each ruling beside its row, and dispatch Task 1. The review
loop remains the net for conflicts that only emerge from implementation.

## The Task Loop

**Batch small same-shape work.** When the plan lists several tasks that are
each a small, independent edit of the same kind — the same one-line fix,
constant change, or field addition across files — do not dispatch one subagent
per task. Compose ONE brief listing every file and its change, send the batch
to a single subagent, and review its diff as one unit. Reserve
one-dispatch-per-task for work needing its own judgment, tests, or review
surface.

Everything you paste into a dispatch prompt — and everything a subagent prints
back — stays resident in your context for the rest of the session and is
re-read on every later turn. Hand artifacts over as files.

**Waiting on subagents:** never poll with short timeouts, and never sit in one
silent open-ended wait. While you have local work — ledger updates, packaging
the next review, reading reports — keep working; results arrive on their own.
When genuinely idle, wait in bounded stretches (five to ten minutes, where
your platform allows), and between stretches post one line of status and
reconcile your live children: list them, chase any that finished without
reporting.

### 1. Dispatch the implementer

Record BASE (`git rev-parse HEAD`) before dispatching — the review package and
fix-round diffs need it.

- **Task brief:** run `scripts/task-brief PLAN_FILE N` — it extracts the task's
  full text to a uniquely named file and prints the path. The brief is the
  single source of requirements; exact values (numbers, magic strings,
  signatures, test cases) appear ONLY there. Never make a subagent read the
  whole plan file.
- **Report file:** name it after the brief (`…/task-N-brief.md` →
  `…/task-N-report.md`) and put it in the dispatch.
- A dispatch describes one task, not the session's history. Do not paste
  accumulated prior-task summaries into later dispatches — a real session's
  dispatch hit 42k chars of which 99% was pasted history. A fresh subagent
  needs its task, the interfaces it touches, and the global constraints.
  Nothing else.
- If an earlier task parked a finding in the area this task touches, carry a
  pointer to that ledger entry.
- Record the implementer's agent identity from the dispatch result — fix round
  1 resumes this agent.
- Never dispatch multiple implementation subagents in parallel (conflicts).

Template: [implementer-prompt.md](implementer-prompt.md)

### 2. Handle the report

**DONE:** generate the review package (`scripts/review-package PLAN_FILE BASE HEAD`
— it prints the unique path it wrote; BASE is the commit you recorded before
dispatching, never `HEAD~1`, which silently drops all but the last commit of a
multi-commit task), then dispatch the task reviewer with that path.

**DONE_WITH_CONCERNS:** read the concerns first. Correctness or scope concerns
get addressed before review; observations ("this file is getting large") get
noted, then proceed to review.

**NEEDS_CONTEXT:** provide the missing context and re-dispatch.

**BLOCKED:** assess the blocker. A context problem → more context, re-dispatch.
Needs more reasoning → re-dispatch with `model: opus`. Too large → break it up.
The plan itself is wrong → rule on the correction, ledger it, re-dispatch with
the ruling carried in the dispatch.

**Never** ignore an escalation or force a retry without changes. If the
implementer said it is stuck, something needs to change. If it asks questions —
before starting or mid-task — answer clearly and completely.

### 3. Review the task

Per-task reviews are task-scoped gates; the broad review happens once at the
end. Never skip the task review, and never accept a report missing either
verdict — spec compliance AND task quality are both required. Implementer
self-review never replaces it.

- Hand the reviewer its diff as a file: run
  `scripts/review-package PLAN_FILE BASE HEAD` and pass the printed path (or,
  without bash: `git log --oneline`, `git diff --stat`, and `git diff -U10`
  for the range, redirected to one uniquely named file). The output never
  enters your context. Use the BASE you recorded — never `HEAD~1`. Never
  dispatch a task reviewer without a diff file.
- **Reviewer inputs:** brief file, report file, review package, plus the
  global constraints that bind the task.
- The global-constraints block is the reviewer's attention lens. Copy binding
  requirements verbatim from the plan or spec: exact values, exact formats,
  stated relationships between components ("same layout as X", "matches Y").
  Process rules are already in the agent — this block is for what THIS
  project's spec demands.
- Do not add open-ended directives ("check all uses", "run race tests if
  useful") without a concrete task-specific reason.
- Do not ask a reviewer to re-run tests the implementer already ran on the
  same code.
- **Do not pre-judge findings.** Never instruct a reviewer to ignore or not
  flag a specific issue. If you believe a finding would be a false positive,
  let it be raised and adjudicate it. If your prompt contains "do not flag,"
  "don't treat X as a defect," "at most Minor," or "the plan chose" — stop:
  you are pre-judging, usually to spare yourself a review loop.

The reviewer may report "⚠️ Cannot verify from diff" items — requirements
living in unchanged code or spanning tasks. These do not block the rest of the
review, but you resolve each one yourself before marking the task complete;
you hold the cross-task context the reviewer lacks. A confirmed gap is a
failed spec review and enters the fix loop.

Template: [task-reviewer-prompt.md](task-reviewer-prompt.md)

### 4. The fix loop

Triggers on spec ❌, any Critical or Important finding, or a ⚠️ item you
confirmed as a real gap.

Two routes leave immediately:

- **Minor findings never enter the loop.** Ledger them
  (`Task <N>: minor (deferred): <one-liner>`) and point the final review at
  that list. A roll-up nobody reads is a silent discard.
- **A plan-mandated finding is yours to rule on** — weigh it against the plan
  text, decide with the spec as binding authority, ledger the ruling before
  acting. Do not dismiss a finding because the plan mandates it, and do not
  dispatch a fix contradicting the plan without a recorded ruling.

Everything else enters the loop. A round is one fix dispatch plus one scoped
re-review. **Two rounds maximum per task.**

**Round 1 — resume the original implementer.** Send the open findings
verbatim. Its context is intact: it knows the task, the code, and its own
choices. If your harness cannot message a live subagent, dispatch a fresh
`sdd-implementer` with the brief path, report-file path, and findings — the
report file is the persistent memory either way.

**Round 2 — dispatch a fresh implementer with `model: opus`,** carrying the
brief path, report-file path, open findings, and this framing: "A prior
implementer attempted this task twice; you own it now. Read the report file
for what was tried." A loop surviving one resume usually means the implementer
cannot see its own problem — fresh eyes and a capability bump in one move.

**Every round:** the implementer fixes, re-runs the tests covering the amended
code, appends its fix report to the same report file, and returns the short
contract. Before re-dispatching the reviewer, confirm the fix report contains
the covering tests, the command, and the output. Name the covering test files
in the fix message — a one-line fix does not need the whole suite.

**The re-review is scoped.** Run `scripts/review-package PLAN_FILE FIX_BASE HEAD`
where FIX_BASE is the head the previous review saw, and dispatch
[re-review-prompt.md](re-review-prompt.md) with the findings list, the brief,
the report file, and the printed diff path. New Critical/Important breakage in
the fix diff joins the open findings. Out-of-scope observations go to the
ledger as deferred minors — they never extend the loop.

**After each round,** append:
`Task <N>: fix round <R>/2 (<X> addressed, <Y> open — <one-liners>; commits <a7>..<b7>)`

Never fix findings yourself in the controller session — your context stays
clean for coordination, and controller fixes skip review.

**The breaker.** When round 2's re-review still leaves findings open, stop
dispatching and adjudicate each one yourself:

- **Reviewer is wrong, or the point is contestable:** park it —
  `Task <N>: parked — <finding> — Ruling: <why the code stands>`. The final
  review sees both sides.
- **Real, but nothing downstream builds on it:** park it the same way, with a
  ruling saying it is real and deferred.
- **Real and load-bearing** — a later task builds on it, or it reveals a plan
  defect: rule on the smallest change that unblocks the dependent work, ledger
  it as `Task <N>: Ruling: <finding> — <what you decided and why>`, and carry
  it into the next task's dispatch. Parking a structural failure silently lets
  every dependent task build on it. Stop only when the defect leaves every path
  forward a guess.

Adjudicate only at the cap — adjudicating earlier to end a loop is pre-judging
with a different name. Every adjudication is a ledger entry; a silent discard
is forbidden.

### 5. Complete the task

When the review comes back clean — or every open finding is parked with a
ruling at the cap — append the completion line in the same message as your
other bookkeeping:

- `Task <N>: complete (commits <base7>..<head7>, review clean)`
- `Task <N>: complete (commits <base7>..<head7>, <K> parked)` after a breaker

Then mark the todo complete and move on. Never move to the next task while the
review has open Critical/Important issues that are neither fixed nor
parked-with-ruling at the cap.

## Final Review

Run `scripts/review-package PLAN_FILE MERGE_BASE HEAD` (MERGE_BASE = where the
branch started, e.g. `git merge-base main HEAD`) and dispatch
`superpowers:code-reviewer` with the printed path, using
[code-reviewer.md](../requesting-code-review/code-reviewer.md). Point it at
the ledger's deferred-minor and parked lines so it can triage what must be
fixed before merge.

If it returns findings, dispatch ONE fix subagent with the complete findings
list — not one fixer per finding. Per-finding fixers each rebuild context and
re-run suites; a real session's final-review fix wave cost more than all its
tasks combined. Then run exactly one scoped re-review of the fix wave.
Adjudicate residuals as in the breaker. There is no second fix wave —
residual load-bearing findings surface to your human partner when
finishing-a-development-branch presents the options.

## Finish

Before deleting anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, breaker adjudications — into your final
message under "Rulings I made", in the order you made them, each with what it
costs if wrong. The list is exhaustive. That list is the only place the
decisions you took on your partner's behalf reach them. A ruling that dies
with the workspace was a decision made in secret.

When the final review is clean and its fixes are merged, delete this plan's
workspace (`rm -rf <workspace>`) — git history is the record now. Sibling
directories belong to other plans; leave them alone.

Use superpowers:finishing-a-development-branch.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Close enough on spec compliance" | Reviewer found spec gaps = not done. Fix, or hit the cap and adjudicate. |
| "I'll fix it myself, dispatching is overhead" | Controller fixes pollute your context and skip review. Resume the implementer. |
| "One more round will converge" | Past the cap, rounds don't converge — the failure is structural. Adjudicate and route. |
| "This finding is obviously wrong, I'll drop it" | You adjudicate only at the cap, and every ruling is a ledger entry. |
| "The fix was small, skip the re-review" | Unreviewed fixes are how regressions land. |
| "Ledger bookkeeping is overhead" | The ledger is what survives compaction. Controllers without one re-dispatched entire completed task sequences. |
| "This task needs a stronger implementer" | The agent tiers are set deliberately. Escalate only where this skill says to, and ledger why. |

## Example Workflow

```
[Setup: worktree verified; plan read once; scripts/sdd-workspace → no ledger, fresh start]
[Pre-flight scan table → ledger. Clean. Create todos.]

Task 1: Hook installation script
[task-brief 1 → dispatch sdd-implementer with brief + report paths + context]
Implementer: "Should the hook install at user or system level?"
You: "User level (~/.config/superpowers/hooks/)"
Implementer: DONE — install-hook command, 5/5 passing, self-review caught a
  missing --force flag, committed.
[review-package BASE HEAD → dispatch sdd-reviewer]
Reviewer: Spec ✅. Issues: none. Task quality: Approved.
[Ledger: Task 1: complete (commits a1b2c3d..d4e5f6a, review clean)]

Task 2: Recovery modes
[task-brief 2 → dispatch sdd-implementer]
Implementer: DONE — verify/repair modes, 8/8 passing, committed.
[review-package BASE HEAD → dispatch sdd-reviewer]
Reviewer: Spec ❌ Missing progress reporting ("report every 100 items").
  Important: magic number (100).
[Fix round 1: resume the implementer with both findings]
Implementer: added progress reporting, extracted PROGRESS_INTERVAL.
  Re-ran test/recovery.test.js — 10/10. Fix report appended.
[review-package FIX_BASE HEAD → scoped re-review]
Re-reviewer: both ADDRESSED (src/recovery.js:41, :7). New breakage: none.
[Ledger: Task 2: fix round 1/2 (2 addressed, 0 open; commits d4e5f6a..b7c8d9e)]
[Ledger: Task 2: complete (commits d4e5f6a..b7c8d9e, review clean)]

[After all tasks: review-package MERGE_BASE HEAD → dispatch code-reviewer]
Final reviewer: all requirements met; deferred minors triaged, none block merge.
[Rulings I made: <list>]
[Delete workspace — the record lives in git now]
Using superpowers:finishing-a-development-branch.
```
