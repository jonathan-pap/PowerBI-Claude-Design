---
name: brand-wordmark
description: Wordmark usage rules and SVG assets for the Contoso placeholder brand. Covers light and dark variants, clear space rules, minimum size, and how to embed in Power BI report headers.
version: 1.0
---

# Brand · Wordmark

## Overview

The wordmark is the primary brand identifier in a Power BI report. It combines a mark (icon) and a logotype (text) in a single lockup. Two variants are provided:

- **Light** — mark in Primary Blue `#0078D4`, logotype in Near-Black `#201F1E`, on a white or light background
- **Dark** — mark in White `#FFFFFF`, logotype in White `#FFFFFF`, on Primary Blue `#0078D4` background (for use in colored page headers)

> **Contoso is a placeholder.** Replace the logotype text and the mark path with your actual brand assets before using in production.

---

## Assets

| File | Variant | Background | Use case |
|------|---------|-----------|---------|
| `assets/wordmark-light.svg` | Light | White / #F3F2F1 | Page header on light backgrounds |
| `assets/wordmark-dark.svg` | Dark | #0078D4 Primary Blue | Colored header zone, dark page backgrounds |

---

## Anatomy

```
[ Mark ]  Logotype
```

| Element | Light variant | Dark variant |
|---------|--------------|-------------|
| Mark (icon) | `#0078D4` | `#FFFFFF` |
| Logotype text | `#201F1E` | `#FFFFFF` |
| Background | Transparent (white context) | `#0078D4` or transparent (dark context) |
| Font | Segoe UI Semibold | Segoe UI Semibold |

The mark is a stylized bar chart — three vertical bars of ascending height — representing data and analytics. It sits to the left of the logotype with 8px spacing between them.

---

## Dimensions

| Property | Value |
|----------|-------|
| Mark height | 24px |
| Mark width | ~20px (proportional) |
| Logotype font size | 18pt Segoe UI Semibold |
| Total lockup height | 32px (with optical padding) |
| Minimum width | 100px |
| Minimum height | 20px (do not scale below this) |

---

## Clear Space

Maintain clear space equal to the height of the mark on all four sides of the wordmark. Never place other visual elements inside the clear space zone.

```
    [  clear space  ]
[c] [ Mark  Name  ] [c]
    [  clear space  ]
```
Where `[c]` = mark height.

---

## Usage Rules

1. **Do not stretch or distort** — scale proportionally only.
2. **Do not recolor** — use only the two approved variants. Never apply arbitrary colors to the mark or logotype.
3. **Do not rotate** — horizontal orientation only.
4. **Do not place on busy backgrounds** — use Light variant on white/light gray, Dark variant on Primary Blue only.
5. **Do not add effects** — no drop shadow, glow, or emboss.
6. **Always use vector SVG** — never rasterize the wordmark at report size.

---

## Placement in Power BI Reports

The wordmark lives in the **page header zone** at the top of every report page.

### Standard placement (light header)

```
x: 24px   y: 12px   height: 32px   width: auto (min 100px)
```

The header zone is typically 48–64px tall. The wordmark sits vertically centered within it.

### Dark header placement

When the page header uses a Primary Blue (`#0078D4`) fill rectangle as background:
- Use `wordmark-dark.svg`
- Same x/y/size rules apply
- The white wordmark provides 7:1 contrast ratio against `#0078D4`

### In Power BI Desktop

Power BI does not have a native "image" element that scales cleanly at all zoom levels. The recommended approach:

1. Insert → Image → select `wordmark-light.svg` or `wordmark-dark.svg`
2. Set the image visual to **Fit** (not Fill or Actual size)
3. Position at x=24, y=12, height=40px, width proportional
4. Lock aspect ratio in the Format pane

Alternatively, embed the wordmark as an SVG DAX measure (see SVG embedding pattern below).

---

## SVG Embedding in DAX Measures

To embed the wordmark inline in a DAX SVG measure (e.g., a dynamic header):

```dax
Page Header SVG =
VAR _svg =
"<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 160 32' width='160' height='32'>"
-- Mark: three ascending bars
& "<rect x='2' y='14' width='5' height='16' rx='1' fill='#0078D4'/>"
& "<rect x='9' y='8' width='5' height='22' rx='1' fill='#0078D4'/>"
& "<rect x='16' y='2' width='5' height='28' rx='1' fill='#0078D4'/>"
-- Logotype
& "<text x='28' y='22' font-family='Segoe UI, Arial, sans-serif' font-size='18' font-weight='600' fill='#201F1E'>Contoso</text>"
& "</svg>"
RETURN _svg
```

For the dark variant, replace `fill='#0078D4'` with `fill='#FFFFFF'` on bars, and `fill='#201F1E'` with `fill='#FFFFFF'` on the text.

---

## Accessibility

- The wordmark SVG includes `role="img"` and `aria-label` for screen readers.
- Contrast ratios: Light variant — logotype `#201F1E` on white = 16:1 ✅; Dark variant — white on `#0078D4` = 7:1 ✅
- Never use the wordmark as the only navigation or orientation cue — page titles serve that role.

---

## Related References

- **colors.md** — Primary Blue `#0078D4`, Near-Black `#201F1E`
- **typography.md** — Segoe UI Semibold, font-weight 600
- **components.md** — Page Header component spec (height, background, title placement)
- **iconography/SKILL.md** — Icon set used alongside the wordmark

---

**Version:** 1.0
**Last Updated:** 2026-05-02
