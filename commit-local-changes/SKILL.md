---
name: commit-local-changes
description: Plan and create atomic local Git commits from a repository working tree. Use only when the user explicitly asks to commit, prepare commits, create commits, or save current changes to Git history. Inspect every change, propose cohesive Conventional Commit groups, require approval of the complete plan before staging, and never push.
---

# Commit Local Changes

Prepare reviewable local commits while preserving unrelated work. Separate
planning from execution: inspect first, present one complete commit plan, and
touch the index or history only after explicit approval of that plan.

## Non-negotiable boundaries

- Start only from an explicit user request to commit or prepare commits. File
  changes alone are never a trigger.
- Treat read-only Git inspection as planning. Do not run `git add`, `git rm`,
  `git commit`, or any command that changes Git state before approval.
- Never push, fetch, pull, amend, rebase, reset, stash, checkout, switch, clean,
  or discard changes as part of this skill.
- Never use `git add .`, `git add -A`, or `git add --all`.
- Preserve user changes, including changes made outside the current session.
- Respect host execution modes, sandbox rules, and approval requirements. Skill
  approval cannot override them.

## Inspect the complete working tree

Base the plan on current repository content, not filenames or session memory.

1. Resolve the repository root and inspect branch and operation state. Stop and
   report an unresolved merge, rebase, cherry-pick, revert, bisect, detached
   HEAD, or other state that makes an ordinary commit unsafe.
2. Run `git status --short --branch --untracked-files=all`.
3. Read both `git diff --cached` and `git diff` for all tracked changes. Use
   `--stat` or `--numstat` as an overview, never as a substitute for content.
4. Inspect every untracked path. Check file type and size before reading it.
   Read text safely; identify binary, generated, large, credential, secret, or
   private-data candidates without printing sensitive values.
5. For every changed path, record the functional change and whether different
   hunks represent unrelated work.
6. Treat pre-existing staged changes as user intent. Preserve them. If their
   current grouping prevents an atomic plan, flag the conflict and ask the user
   to authorize or perform index reorganization; never unstage silently.

Every visible staged, unstaged, deleted, and untracked path must appear exactly
once in the plan or in the explicit leave-uncommitted list.

## Form atomic commit groups

Group by one reviewable intent, not by filename, directory, or edit chronology.

Keep implementation, directly dependent call sites, mechanical support, and
direct tests together. Split unrelated fixes, independent features, formatting,
refactors, documentation, exploratory code, and drive-by changes when they can
be reviewed or reverted independently.

Use this test: reverting one proposed commit should cleanly undo one thing. If
not, split it. When intent remains ambiguous after inspection, default to a
split and ask rather than guessing.

If one file mixes unrelated work, identify the exact hunks in separate commit
groups and plan hunk-level staging. Do not assign the whole file to either
group.

## Write Conventional Commit messages

Follow Conventional Commits 1.0.0:

```text
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

- Use `feat` for a new feature and `fix` for a bug fix.
- Prefer `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`,
  and `revert` for their usual meanings. Other lowercase noun types are valid.
- Use an optional lowercase noun scope only when it adds useful context.
- Use a lowercase imperative description: `fix: handle missing sample IDs`.
- Limit the full subject, including type and scope, to 50 characters.
- Do not end the subject with a period.
- Add a body when non-obvious context is useful. Explain what changed and why,
  not line-by-line implementation details.
- Separate the body from the subject with one blank line and wrap body lines at
  72 characters. Multiple body paragraphs are valid.
- Mark breaking changes with `!`, a `BREAKING CHANGE:` footer, or both.
  `BREAKING-CHANGE:` is an equivalent footer token.
- Use Git-style footer tokens for issues and acknowledgements when relevant.

Before committing, review the exact message against every rule above. Count the
subject and body line lengths explicitly. Imperative mood and whether a body is
warranted require semantic judgment rather than filename-based assumptions.

## Present one complete plan

Use this shape for every requested operation:

```text
Proposed commit plan (N commits):

Commit 1: <short label>
Files/hunks: <exact paths and partial-file hunk descriptions>
Reason: <why this is one cohesive unit>
Message:
  <complete subject>

  <complete optional body and footers>

...

Summary: N commits proposed. <All changes accounted for, or exact paths left
uncommitted with reasons.>
```

Ask for approval of the entire plan. Accept only an unambiguous response that
refers to the current complete plan, such as `approve` or `execute this plan`.
Silence, unrelated follow-up, or approval of only one unsettled part is not full
approval.

If the user rewords, merges, splits, reorders, or drops anything, regenerate and
present the complete replacement plan. Do not execute a subset while the rest
is still being negotiated.

## Execute an approved plan

Immediately before touching Git state, repeat status and diff inspection. If
any staged, unstaged, deleted, or untracked content differs from the approved
plan, invalidate approval and produce a new complete plan.

For each approved commit in order:

1. Stage whole files with `git add -- <exact-paths>` only when every change in
   each file belongs to that commit.
2. Use `git add -p -- <file>` for approved partial-file hunks. If hunk selection
   cannot be performed reliably, stop and ask; never stage the whole file as a
   shortcut.
3. Inspect `git diff --cached --stat` and the complete `git diff --cached`.
   Confirm the index matches only the current approved commit.
4. Run `git diff --cached --check`.
5. Validate the exact approved message, then commit with that exact message.
   Do not use `--no-verify`; allow repository hooks to run.
6. Verify the new commit with `git show --stat --oneline --decorate -1` and
   re-run status before moving to the next commit.

If a hook fails, modifies content, or the working tree changes during the
series, stop. Report commits already created and remaining changes without
rolling back, amending, or continuing under stale approval.

## Report completion

List each created commit hash and subject. Report every remaining staged,
unstaged, deleted, or untracked path and why it remains. State explicitly that
no push was performed.
