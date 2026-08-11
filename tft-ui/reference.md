# TFT UI — draw sketches

Library-agnostic layout notes. Adafruit GFX snippets are examples only.

## Parameter row — name left, value right

```
| pad | NAME .................... VALUE | pad |
| pad | LEAK ..................... 10s | pad |
| pad | WIFI ..................... [✓] | pad |
```

- **Name:** left-aligned at the row’s left pad.
- **Value:** right-aligned at the row’s right pad (text value or checkbox).
- Same font/baseline for name and value unless the project defines otherwise.
- Do not draw `"NAME VALUE"` as one left-aligned string.

```cpp
constexpr int kPadX = 6;

void drawParamRow(int y, int h, const char *name, const char *value, uint16_t fg, uint16_t bg) {
    fillRect(0, y, TFT_WIDTH, h, bg);
    setFont(...);
    setTextSize(1);
    setTextColor(fg);

    int16_t x1, y1;
    uint16_t tw, th;
    // name — left
    getTextBounds(name, 0, 0, &x1, &y1, &tw, &th);
    int baseline = y + (h + (int)th) / 2;
    setCursor(kPadX - x1, baseline);
    print(name);

    // value — right
    getTextBounds(value, 0, 0, &x1, &y1, &tw, &th);
    setCursor(TFT_WIDTH - kPadX - (int)tw - x1, baseline);
    print(value);
}
```

## BACK row layout

```
| pad | [up icon] | <-- 15 px --> | BACK                  |
```

- Icon: native resolution, drawn 1:1 (no pixel replication).
- Unfocused `BACK` text and icon: light red (`0xFD14` RGB565 default).
- Focused / editing styling follows the project’s focus chrome; keep `BACK` identifiable as navigation.

Adafruit GFX example:

```cpp
constexpr int kPadX = 6;
constexpr int kBackIconW = 12;
constexpr int kBackIconH = 12;
constexpr int kBackIconGap = 15;  // hard rule
constexpr uint16_t kBackFg = 0xFD14;

drawBitmap(kPadX, iconY, upLevelIcon, kBackIconW, kBackIconH, fg);
setCursor(kPadX + kBackIconW + kBackIconGap - x1, baseline);
print("BACK");
```

## Dotted separator (item height < 22 px)

- 1 px high, dark-dark gray (`0x2104` RGB565 default).
- Draw on the bottom pixel row of the item band (or between bands).
- Pattern: every other pixel (or equivalent dotted style).

```cpp
constexpr uint16_t kSepColor = 0x2104;

void drawDottedSeparator(int y, int width) {
    for (int x = 0; x < width; x += 2) {
        drawPixel(x, y, kSepColor);
    }
}
```

Skip when item height ≥ 22 px.

## Rounded Confirm / Cancel

- Left and right ends rounded (`fillRoundRect` or equivalent).
- Confirm: fg `0xCFF7`, bg `0x0320`.
- Cancel: fg `0xFCD3`, bg `0x9800`.

```cpp
constexpr uint16_t kConfirmFg = 0xCFF7;
constexpr uint16_t kConfirmBg = 0x0320;
constexpr uint16_t kCancelFg  = 0xFCD3;
constexpr uint16_t kCancelBg  = 0x9800;

// radius ≈ half button height for pill-like left/right ends
fillRoundRect(x, y, w, h, h / 2, bg);
setTextColor(fg, bg);
// center label baseline inside the button
```

## Binary parameter checkbox

```
| pad | LABEL ......................... | [ ] |   OFF
| pad | LABEL ......................... | [✓] |   ON   (4 px bird ✓; green unfocused, black focused/edit)
```

- Do **not** draw `ON` / `OFF` text as the value.
- Box size: choose a fixed native size that fits the row (example 14×14); draw 1:1, no scaling.
- Place the checkbox on the trailing side of the row with a small right pad; leave enough pad so an oversized ✓ does not clip.
- Outline: 1 px rectangle in the **row foreground** color.
- Checked: draw a **bird-like check mark** (✓) with short down-right then longer up-right strokes. Stroke weight must be **bold 4 px** (four parallel 1 px lines, or equivalent). Color: **green** (`0x07E0`) when **unfocused**; **black** (`0x0000`) when **focused or in edit mode**. The mark must be **a little larger than the box**. Do **not** use a filled inner square or ON/OFF text.

```
Check geometry relative to 14×14 box origin (local coords; overflows box):
  short: (-1,6) → (5,12)
  long:  (5,12) → (15,-1)
```

```cpp
constexpr int kPadX = 6;
constexpr int kCheckSize = 14;
constexpr int kCheckRightPad = 8;  // room for oversized ✓
constexpr int kCheckStrokePx = 4;
constexpr uint16_t kCheckMarkGreen = 0x07E0;
constexpr uint16_t kCheckMarkOnFocus = 0x0000;  // black when focused or editing

void drawThickLine(int x0, int y0, int x1, int y1, uint16_t color, int thickness) {
    // Offset along the approximate normal; for ✓ arms, vertical offsets work well.
    const int half = thickness / 2;
    for (int i = 0; i < thickness; ++i) {
        const int dy = i - half;
        drawLine(x0, y0 + dy, x1, y1 + dy, color);
    }
}

void drawBirdCheck(int boxX, int boxY, uint16_t checkFg) {
    drawThickLine(boxX - 1, boxY + 6, boxX + 5, boxY + 12, checkFg, kCheckStrokePx);
    drawThickLine(boxX + 5, boxY + 12, boxX + 15, boxY - 1, checkFg, kCheckStrokePx);
}

void drawCheckbox(int rowY, int rowH, bool checked, bool selected, bool editing, uint16_t rowFg) {
    const int boxX = TFT_WIDTH - kCheckRightPad - kCheckSize;
    const int boxY = rowY + (rowH - kCheckSize) / 2;
    drawRect(boxX, boxY, kCheckSize, kCheckSize, rowFg);
    if (checked) {
        const uint16_t mark =
                (selected || editing) ? kCheckMarkOnFocus : kCheckMarkGreen;
        drawBirdCheck(boxX, boxY, mark);
    }
}
// label drawn separately on the left; never append " ON"/" OFF"
```

## Fonts

Never upscale. On Adafruit GFX: `setTextSize(1)` and a sized GFXfont (e.g. FreeSansBold9pt7b), never `setTextSize(n≠1)` to enlarge text.

## Partial redraw

- Don't call full-screen clear in the hot path.
- Clear only the bar (row) being updated (`fillRect` for that row’s y/height), then redraw its contents.
- Full-screen clear only on first paint or when the whole view changes (e.g. menu level switch).
