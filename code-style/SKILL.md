---
name: code-style
description: >-
  Naming and code-style conventions for all projects. Use always when writing,
  editing, reviewing, or refactoring code in any language; when adding variables,
  parameters, fields, or locals; or when the user mentions CodeStyle / naming.
---

# CodeStyle

Applies to **all** projects. Follow when creating or changing code.

## Variable naming

Never name variables as "s", "a", "_var" and so on. Always use meaningful names.

```cpp
// BAD practice
GlobalSettings *s;

// GOOD practice
GlobalSettings *globalSettings;
```

### Also avoid

- Single-letter names except conventional loop indices (`i`, `j`, `k`) in tiny scopes
- Cryptic abbreviations (`tmp2`, `val`, `obj`, `ptr`, `data1`)
- Leading-underscore locals used as the real name (`_var`, `_s`, `_settings`)
- Type-initial acronyms that hide meaning (`gs` for GlobalSettings, `sm` for SettingsManager) when a clear camelCase/PascalCase name fits the project

### Prefer

- Names that state role or content: `globalSettings`, `settingsManager`, `mqttController`, `heaterCoreLiters`
- Match existing project casing (camelCase / snake_case / PascalCase) — keep names meaningful either way
