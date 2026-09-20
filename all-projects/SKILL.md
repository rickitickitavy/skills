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
- If the current branch is "master" or "develop", propose to create the new branch before committing the work. Do not commit on those branches until the user accepts or refuses the new branch.

## Questions vs tasks

| User intent | Allowed |
|-------------|---------|
| Question / explanation / “why” / status | Answer only. No file, shell, or git changes. |
| Explicit task (“do X”, “fix Y”, “create Z”) | Act only within that request. |

When unsure whether the message is a question or a task, treat it as a question and answer only.

## New entity properties (self-check)

After you add any property on a persisted entity, you MUST follow its **full life cycle** before you say it is added. Persistence alone is not enough. You very often drop the property on list sync, reboot, or UI surfaces.

### Persistence

1. **Single-item save and read** — create/update one entity, persist, load that same entity back; the property round-trips.
2. **List operations** — list APIs, dumps, pull/replace, import/export, and any bulk sync keep the property (including after reboot or replacing the list from another source).

### UI (required)

Trace the property through **every screen that shows that entity**, not only the place you just edited:

- **Tables / lists** — every visual table or list row (main entity table, search/found table, and any other list of the same entity).
- **Dialogs** — add, edit, settings, confirm, and any other dialog that opens that entity (readonly or editable as the design requires).
- **Reload** — after save and after reboot, those same tables and dialogs still show the stored value, not a blank/default.

Do not claim the property is added if any table or dialog omits it, or if the UI shows a default after a correct save. Fix the gaps first.
