---
name: develop-plan
description: Hold a repository-aware planning discussion without automatically drafting, reproducing, revising, or implementing the plan. Use when the user explicitly invokes this skill to behave as in Codex Plan mode while keeping ordinary comments and questions as discussion until the user explicitly asks to draft or update the plan or to begin implementation.
---

# Develop Plan

Behave as in Codex Plan mode. Keep discussion, plan output, and implementation
as separate user-controlled stages.

## Discuss by default

- Take any non-mutating action useful for understanding the task and answering
  accurately, following the same exploration and permission boundaries as Codex
  Plan mode.
- Use the current repository as primary context. Inspect relevant files,
  history, configuration, documentation, and existing patterns as needed.
- Ask questions, answer questions, evaluate suggestions, and compare approaches
  without treating the conversation as authorization to change anything.
- Do not implement changes while discussing. Respect any stricter host mode,
  sandbox, permission, or user instruction.

## Gate plan output explicitly

- Do not draft the first formal plan until the user explicitly asks to draft,
  write, or produce it.
- After producing a plan, return to discussion by default.
- Treat comments, questions, agreement, criticism, and proposed alternatives as
  discussion. Do not reproduce, revise, replace, or append the plan in response.
- Revise the plan only when the user explicitly asks to update, revise, rewrite,
  replace, or redraft it.
- When intent is ambiguous, continue discussing instead of emitting a plan.

## Gate implementation explicitly

- Do not infer implementation authorization from an approved approach, a
  completed plan, or positive feedback.
- Begin implementation only when the user explicitly asks to implement, apply,
  execute, build, fix, or make the changes.
- On an explicit implementation request, leave this discussion workflow and
  follow the current host permissions and any applicable implementation skills.
