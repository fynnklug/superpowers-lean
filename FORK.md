# Lean fork of Superpowers

Fork of [obra/superpowers](https://github.com/obra/superpowers) at `v6.3.0`
(`b36e0829`), trimmed for token cost. Branch `lean`; upstream tracked as the
`upstream` remote.

## What changed

- **`agents/`** — the reason for the fork. Upstream ships no agent definitions,
  so every SDD dispatch named `general-purpose`, which inherits the session's
  model *and* effort level. The Agent tool has a `model` parameter but no
  `effort` parameter, so agent frontmatter is the only place a subagent's
  reasoning effort can be lowered at all.
- **Dispatch templates** carry only task-specific content; standing contracts
  live in the agent bodies, so the controller stops pasting them into its own
  context on every dispatch.
- **`sdd-implementer` denies the `Agent` tool**, replacing prose that asked
  implementers not to spawn reviewers.
- **Fix-loop cap** 5 rounds → 2; round 2 escalates via `model: opus`.
- **Both SDD graphviz digraphs dropped** — they restated the adjacent prose.
- **Brainstorming tie-break reversed** to the lighter path with a named
  upgrade trigger.
- `test-driven-development` deliberately untouched.

## Updating from upstream

```bash
git fetch upstream
git log --oneline lean..upstream/main        # what's new
git diff upstream/main..lean --stat          # what this fork owns
git merge upstream/main                      # resolve; the files below always conflict
```

Expect conflicts in exactly these, and re-apply the fork's intent rather than
taking either side wholesale:

    skills/subagent-driven-development/{SKILL.md,implementer-prompt.md,
                                        task-reviewer-prompt.md,re-review-prompt.md}
    skills/{brainstorming,using-superpowers}/SKILL.md

New upstream dispatch sites appear as `Subagent (general-purpose):` — repoint
them at an agent in `agents/`, or they will silently inherit the session model.

## Reinstalling after an edit

The marketplace source is this directory, but Claude Code **copies** the plugin
into `~/.claude/plugins/cache/superpowers-lean/superpowers/<version>/`. Edits
here do not take effect until you refresh that copy:

```bash
claude plugin marketplace update superpowers-lean
claude plugin uninstall superpowers@superpowers-lean
claude plugin install superpowers@superpowers-lean -y
```

Bump `version` in both `.claude-plugin/plugin.json` and
`.claude-plugin/marketplace.json` when the change is worth tracking.

## Measuring the effect

`tests/claude-code/analyze-token-usage.py` breaks a session transcript down by
main session vs. individual subagents:

```bash
python tests/claude-code/analyze-token-usage.py ~/.claude/projects/<project>/<session>.jsonl
```
