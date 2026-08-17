---
name: all-projects
description: >-
  Hard interaction rules that apply to every project and every conversation.
  Use always: before any edit, restore, git operation, or tool action; and
  whenever the user asks a question, requests an explanation, or seeks status.
---

# ALL PROJECTS

Applies to **all** projects. Read and follow before acting.

## Hard rules

- NEVER use git to restore any changes.
- NEVER restore a whole file without asking me first.
- NEVER EVER do an action as a reaction to any question.
- If I ask any question, you MUST only answer — no edits, no tool side effects, no restores, no git mutations.

## Questions vs tasks

| User intent | Allowed |
|-------------|---------|
| Question / explanation / “why” / status | Answer only. No file, shell, or git changes. |
| Explicit task (“do X”, “fix Y”, “create Z”) | Act only within that request. |

When unsure whether the message is a question or a task, treat it as a question and answer only.
