# TFT UI — draw sketches

Library-agnostic layout notes. Adafruit GFX snippets are examples only.

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

## Fonts

Never upscale. On Adafruit GFX: `setTextSize(1)` and a sized GFXfont (e.g. FreeSansBold9pt7b), never `setTextSize(n≠1)` to enlarge text.

## Partial redraw

- Don't call full-screen clear in the hot path.
- Clear only the bar (row) being updated (`fillRect` for that row’s y/height), then redraw its contents.
- Full-screen clear only on first paint or when the whole view changes (e.g. menu level switch).
