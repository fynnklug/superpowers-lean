---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring skill invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, ignore this skill.
</SUBAGENT-STOP>

## The Rule

**Invoke relevant or requested skills BEFORE any response or action** —
including clarifying questions, exploring the codebase, or checking files. If
a skill turns out wrong for the situation, you don't have to use it — but you
check first. If there is even a 1% chance a skill applies, invoke it.

**Before entering plan mode:** if you haven't brainstormed, invoke the
brainstorming skill first.

Then announce "Using [skill] to [purpose]" and follow it. If it has a
checklist, create a todo per item.

## Skill Priority

Process skills come first — they set the approach; implementation skills then
carry it out.

- "Let's build X" → superpowers:brainstorming, then implementation skills
- "Fix this bug" → superpowers:systematic-debugging, then domain skills

## Red Flags

These thoughts mean you are rationalizing — check for skills anyway:

| Thought | Reality |
|---------|---------|
| "This is just a simple question" / "This doesn't count as a task" | Questions and actions are both tasks. |
| "Let me explore / gather context first" | Skills tell you HOW to explore. Check first. |
| "I remember this skill" | Skills evolve. Read the current version. |
| "The skill is overkill here" | Skills scale their ceremony to the task. Invoke it and let it. |

## Platform Adaptation

If your harness appears here, read its reference file: Codex
(`references/codex-tools.md`), Pi (`references/pi-tools.md`), Antigravity
(`references/antigravity-tools.md`), Hermes Agent
(`references/hermes-tools.md`).

## User Instructions

User instructions (CLAUDE.md, AGENTS.md, direct requests) take precedence over
skills, which in turn override default behavior. Only skip a skill workflow
when your human partner has explicitly told you to.
