---
name: web-console
description: >-
  Builds ESP/LittleFS web consoles in the boiler-controller visual language:
  left sidebar main sections, folder-style inner tabs, purple accent, 44px
  fields, upload cards, log viewer. Use when adding or changing a device web
  UI, index.html, all.css, AsyncWebServer pages, settings tabs, OTA/log
  screens, or when the user mentions web console, web UI, or SoftAP config page.
---

# Web console (shared chrome)

Source of truth: `boiler-controller` (`data/index.html` + `data/css/all.css`). Copy the chrome, not the boiler SVG/domain forms.

Use with CodeStyle for identifiers. Do not invent a second look (Material, Bootstrap, dark theme) unless the user asks.

## Layout

```
+-- app-shell ----------------------------------+
| sidebar-nav        | main-panel               |
|  sidebar-title     |  section-title           |
|  sidebar-btn ...   |  [inner tabs]            |
|                    |  section-panel.active    |
+-----------------------------------------------+
```

- One page. No SPA framework. Serve from LittleFS: `/index.html`, `/css/all.css`.
- **Main sections:** left **vertical tab strip** (`.sidebar-nav` / `.sidebar-btn`). Active button shares the panel background and hides its right border.
- **Inner groups** (Settings/System/etc.): **folder tabs** (`.tabs` / `.tab-btn` / `.tab-panel`). Active tab sits on the content surface (`border-bottom` matches surface).
- Only one `.section-panel.active` and one `.tab-panel.active` per strip.
- Desktop: `html, body` full viewport, `overflow: hidden`; the active panel scrolls.
- Narrow (`max-width: 600px`): sidebar becomes a horizontal folder strip; hide `.sidebar-title`.

Markup names: `data-section`, `section-<name>`, `data-tab`, `tab-<name>` or `system-tab-<name>`. JS toggles class `active` only (see boiler `showMainSection` / `showSettingsTab` / `showSystemTab`).

## Tokens (do not restyle)

Copy from [chrome.css](chrome.css). Required `:root`:

| Token | Value |
|-------|--------|
| `--accent` | `#290682` |
| `--accent-soft` | `#ede8f8` |
| `--bg` | `#e8e8e8` |
| `--surface` | `#ffffff` |
| `--border` | `#c8c8c8` |
| `--text` | `#1a1a1a` |
| `--muted` | `#5a5a5a` |
| `--radius` | `8px` |
| `--gap` | `0.75rem` |
| `--sidebar-width` | `9.5rem` |

Idle tabs/sidebar: background `#dcdcdc`, muted text. Hover (not active): accent text on `--accent-soft`. Active: accent text, `--surface`, weight 700.

Font: `system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif` at 16px. Shell padding: `2cm 0.5cm 1cm 2cm`.

## Controls

- Touch target **min-height 44px** on sidebar buttons, folder tabs, inputs, selects, `.btn`, password toggle.
- **Fields:** `.field` card (border, radius, padding). Label 0.875rem / 600. Inputs on `--bg`, focus outline 2px `--accent`.
- **Checkbox:** `.field-check` row; box 1.35rem; `accent-color: var(--accent)`.
- **Password:** `.password-row` + `.password-toggle` (“Show” / “Hide”).
- **Primary button:** `.btn.btn-primary` (accent fill, white text). **Secondary:** `.btn.btn-secondary`.
- **Save banners:** `.save-error` / `.save-success` (red / green boxes as in chrome.css).
- **OTA:** `.upload-card` + `.upload-title` + `.progress-bar` + `.upload-status`.
- **Log:** readonly `textarea.log-viewer` (monospace, `white-space: pre`) + Refresh `.btn-secondary`.
- **Section headings:** `.section-title` / `.upload-title` / `.group-title` use `--accent`.

## Do / don’t

- Do keep `role="navigation"` / `tablist` / `tab` / `tabpanel`.
- Do poll status only while the Status section is active (boiler `setStatusPolling`).
- Don’t add hamburger menus, cards-as-nav, or a second CSS framework.
- Don’t copy boiler-only SVG (`.svg-*`) into other products.
- Don’t change tokens to “match branding” unless the user asks.

## New project checklist

1. Copy [chrome.css](chrome.css) to `data/css/all.css`.
2. Build `data/index.html`: `app-shell` → sidebar of product sections → `main-panel` sections.
3. Inner folder tabs only where the product has groups (e.g. System: Update, Log).
4. Wire `/` to LittleFS `index.html`; static `/css/*` as files. Template processors optional.
