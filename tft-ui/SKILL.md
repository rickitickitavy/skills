---
name: tft-ui
description: >-
  Builds multilevel TFT/screen UI menus and dialogs: navigate items, enter/exit
  field edit, change values, BACK row with up-level icon, dotted separators when
  rows are short, and rounded Confirm/Cancel buttons. Controller-agnostic (map
  rotary, buttons, D-pad, etc. to the same actions). Use when implementing or
  changing TFT/screen menus, parameter lists, submenu navigation, or dialog
  buttons on any embedded display UI.
---

# TFT / screen UI

Controller-agnostic conventions for multilevel menus, parameter editing, and dialog buttons on TFT / embedded screens. Not tied to any MCU or graphics library.

## Interaction (map hardware to these actions)

| Action | Browse mode | Edit mode |
|--------|-------------|-----------|
| **Navigate** | Move focus between parameters / menu items | Do not move focus |
| **Activate** (single click / confirm key) | Enter field edit mode, or activate navigation items (open submenu / BACK) | Exit field edit mode |
| **Adjust** | — | Change focused field value |

Example mappings (use what the project has):

- Rotary encoder: Navigate/Adjust = rotate; Activate = button single click
- D-pad / buttons: Navigate = up/down; Adjust = left/right or up/down; Activate = OK/Enter
- Touch: Navigate = tap item; Adjust = drag/step controls; Activate = tap focused value / Done

Edit mode is local to the focused editable field. Navigation items (submenu entries, `BACK`) use **Activate** only; they do not stay in value-edit mode.

## Multilevel menus — BACK

- Every submenu level lists **`BACK` as the first item**.
- Left of `BACK`: small native-resolution **"go to up level"** icon (draw 1:1; no bitmap upscaling).
- **15 px** horizontal gap between icon and `BACK` label.
- `BACK` label (and matching icon when unfocused) uses **light red** font color.

## Item separators

- If menu **item height is less than 22 px**, draw a **1 px high dark-dark gray dotted line** between items.
- If item height is ≥ 22 px, do not require these separators.

## Confirm / Cancel buttons

- **Confirm:** light-light-green font on dark green background; left and right ends rounded.
- **Cancel:** light-light red font on dark red background; left and right ends rounded.

## Display hard rules

- **Never upscale fonts.** Choose a properly sized font; do not scale a smaller face (on Adafruit GFX: `setTextSize(1)` + sized GFXfont only).
- Never scale icons/bitmaps at draw time; draw native-resolution glyphs 1:1.
- **Don't clear the whole screen.** Clear / redraw only the bar (row) that will be updated. Full-screen clear is allowed only on first paint or when the entire view must change (e.g. menu level switch).

## Palette defaults

Prefer RGB565 on 16-bit TFTs. Projects may translate to RGB888 / other formats; keep the same named roles. Hex may be overridden per project; roles, 15 px gap, rounded Confirm/Cancel, no font upscale, and the separator threshold are hard rules.

| Role | RGB565 | Notes |
|------|--------|--------|
| BACK text / icon (unfocused) | `0xFD14` | light red |
| Confirm fg | `0xCFF7` | light-light green |
| Confirm bg | `0x0320` | dark green |
| Cancel fg | `0xFCD3` | light-light red |
| Cancel bg | `0x9800` | dark red |
| Separator (dotted, 1 px) | `0x2104` | dark-dark gray |
| Icon↔label gap | `15` px | hard rule |
| Separator threshold | item height `< 22` px | hard rule |

## Additional resources

- Draw sketches (BACK row, separator, rounded button): [reference.md](reference.md)
