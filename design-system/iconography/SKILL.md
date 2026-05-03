---
name: iconography
description: Icon system for Power BI design. Fluent line style — 20px viewBox, 1.5px stroke, currentColor. Covers naming conventions, sizing grid, semantic color application, inline SVG embedding in DAX measures, full asset index, and accessibility requirements.
version: 2.0
---

# Iconography

## Overview

The iconography system provides a consistent set of Fluent-style line icons for use in Power BI reports and SVG DAX measures. Icons follow Microsoft's Fluent UI System aesthetic: clean outlines, rounded caps, 1.5px stroke weight, and `currentColor` so the icon inherits its color from the context it is placed in.

**Icons are never used as the sole communication channel.** Always pair an icon with a text label, data value, or tooltip.

---

## Style Specification

| Property | Value |
|----------|-------|
| Style | Fluent line (outline) |
| ViewBox | `0 0 20 20` |
| Stroke | `currentColor` |
| Stroke-width | `1.5` |
| Stroke-linecap | `round` |
| Stroke-linejoin | `round` |
| Fill | `none` (line icons) or semantic color (status dots only) |

**`currentColor` means the icon inherits its color from the CSS `color` property (or SVG `fill`/`stroke` on a parent element).** This makes it trivial to colorize icons in DAX SVG measures — set the color on the wrapping `<svg>` element and all strokes inherit it.

### What NOT to do

```xml
<!-- ✗ Hardcoded fill — can't be recolored -->
<path fill="#107C10" d="M10 4 L10 16..."/>

<!-- ✓ currentColor — inherits from parent -->
<svg color="#107C10">
  <path stroke="currentColor" stroke-width="1.5" fill="none" d="M10 4 L10 16..."/>
</svg>
```

---

## Naming Convention

```
{concept}-{modifier}.svg
```

| Segment | Rules | Examples |
|---------|-------|---------|
| concept | Lowercase noun or verb | `arrow`, `chev`, `caret`, `dot` |
| modifier | Lowercase direction abbreviation or color role | `up`, `dn`, `lt`, `rt`, `flat` |

**Use the short forms:** `dn` not `down`, `lt` not `left`, `rt` not `right`. This aligns with the design system tool display names.

---

## Sizing Grid

All icons use a **20×20 viewBox** as the master (Fluent UI System standard for the 20px size). Scale via `width` and `height` attributes only — never change the viewBox.

| Render size | Use case |
|-------------|---------|
| 16px | Inline with body text (12pt), table cells |
| 20px | Default — inline with labels, slicer values, KPI labels |
| 24px | Standalone status indicators, KPI card trend icons |

```xml
<!-- 16px render -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" width="16" height="16">...</svg>

<!-- 20px render (default) -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" width="20" height="20">...</svg>
```

---

## Color Application

Icons use `currentColor`. Apply semantic color by setting `color` or wrapping the icon in a parent with the relevant fill.

| Use case | Color token | Hex |
|---------|-------------|-----|
| Positive trend (arrow-up) | `--color-good` | `#107C10` |
| Negative trend (arrow-dn) | `--color-bad` | `#D13438` |
| Flat / no change (flat) | `--color-neutral` | `#605E5C` |
| Warning | `--color-warning` | `#FFB900` |
| Success / confirmed (check) | `--color-good` | `#107C10` |
| Informational (info) | `--color-primary` | `#0078D4` |
| UI chrome (filter, search, etc.) | `--color-text-primary` | `#201F1E` |
| Status dots | Semantic fill color (see dot assets) | — |

**Status dots (dot-green, dot-amber, dot-red)** are the only filled icons in the set. They use a solid `fill` (no stroke) because a 1.5px stroke circle at small sizes is too thin to read at a glance.

---

## Asset Index

All icons are in `iconography/assets/`. Line icons use `currentColor`; status dots use semantic fill.

### Line Icons (Fluent style · stroke · currentColor)

| File | Glyph | Use case |
|------|-------|---------|
| `arrow-up.svg` | ↑ | Positive trend, growth, increase |
| `arrow-dn.svg` | ↓ | Negative trend, decline, decrease |
| `flat.svg` | → | No change, flat trend |
| `caret-dn.svg` | ▾ | Dropdown toggle, expand |
| `chev-lt.svg` | ‹ | Navigate back, previous |
| `chev-rt.svg` | › | Navigate forward, next |
| `filter.svg` | ⊽ | Filter panel toggle |
| `refresh.svg` | ↺ | Refresh data, reload |
| `search.svg` | ⌕ | Search input |
| `warning.svg` | ⚠ | Caution, alert — apply `color="#FFB900"` |
| `check.svg` | ✓ | Success, confirmed — apply `color="#107C10"` |
| `info.svg` | ⓘ | Informational — apply `color="#0078D4"` |
| `grid.svg` | ⊞ | Grid/table view toggle |
| `export.svg` | ↑□ | Export data, share |

### Status Dots (filled · semantic color)

| File | Fill | Use case |
|------|------|---------|
| `dot-green.svg` | `#107C10` | Healthy, on track |
| `dot-amber.svg` | `#FFB900` | Warning, at risk |
| `dot-red.svg` | `#D13438` | Critical, off track |

---

## Embedding Icons in SVG DAX Measures

Power BI SVG measures run in a sandboxed environment. **No external image references load.** Icons must be inlined as SVG path data.

### Pattern: currentColor in DAX

```dax
Trend Icon =
VAR _gap = [$ Revenue] - [$ Revenue Target]
VAR _color =
    IF( _gap > 0, "#107C10",
    IF( _gap < 0, "#D13438", "#605E5C" ))
VAR _path =
    IF( _gap > 0,
        -- arrow-up path
        "M10 15 L10 5 M6 9 L10 5 L14 9",
        IF( _gap < 0,
        -- arrow-dn path
        "M10 5 L10 15 M6 11 L10 15 L14 11",
        -- flat path
        "M4 10 L16 10 M12 6 L16 10 L12 14"
        )
    )
VAR _svg =
    "<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20' width='20' height='20'>"
    & "<path d='" & _path & "' stroke='" & _color & "' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round' fill='none'/>"
    & "</svg>"
RETURN _svg
```

### Rules for SVG in DAX

1. **Single quotes inside the SVG string** — DAX uses double-quoted strings; XML attributes inside use single quotes.
2. **Always set `width` and `height`** — Power BI renders at wrong size without explicit dimensions.
3. **One `<svg>` root element** per measure return value.
4. **No external `href` or `src`** — sandbox blocks all external references.
5. **No `<style>` with `@import` or `url()`** — same sandbox restriction.
6. **Font stack for any text elements:** `font-family='Segoe UI, Arial, Calibri, sans-serif'`

### Combining Icon + Text

```dax
KPI Badge =
VAR _gap = [$ Revenue] - [$ Revenue Target]
VAR _color = IF( _gap >= 0, "#107C10", "#D13438" )
VAR _path = IF( _gap >= 0,
    "M10 15 L10 5 M6 9 L10 5 L14 9",
    "M10 5 L10 15 M6 11 L10 15 L14 11" )
VAR _label = FORMAT( ABS( _gap ), "$#,0" )
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 120 20' width='120' height='20'>"
& "<svg x='0' y='0' viewBox='0 0 20 20' width='20' height='20'>"
& "<path d='" & _path & "' stroke='" & _color & "' stroke-width='1.5' stroke-linecap='round' stroke-linejoin='round' fill='none'/>"
& "</svg>"
& "<text x='24' y='14' font-family='Segoe UI, Arial, sans-serif' font-size='12' fill='" & _color & "'>" & _label & "</text>"
& "</svg>"
```

---

## Accessibility Requirements

1. **Never icon-only** — always pair with a text label, value, or tooltip.
2. **Add `<title>` for standalone icons:**
   ```xml
   <svg viewBox="0 0 20 20" width="20" height="20" role="img" aria-label="Positive trend">
     <title>Positive trend</title>
     <path d="M10 15 L10 5 M6 9 L10 5 L14 9"
           stroke="currentColor" stroke-width="1.5" stroke-linecap="round" fill="none"/>
   </svg>
   ```
3. **Minimum render size:** 16px — below this Fluent line icons lose legibility.
4. **Contrast:** `currentColor` inherits from context. Ensure the applied color meets WCAG AA (4.5:1) against background.
5. **Status dots:** always pair with a text label — a dot communicates status via color only.

---

## Substitution Note

These icons are **Fluent-style placeholders**. For production parity, substitute with the official **Microsoft Fluent UI System Icons** library (MIT license, available at github.com/microsoft/fluentui-system-icons). Match by name and size variant (20px Regular).

---

## Adding New Icons

1. Follow naming convention: `{concept}-{modifier}.svg`
2. ViewBox `0 0 20 20`, stroke `currentColor`, stroke-width `1.5`, fill `none`
3. Design system colors only
4. Add to the Asset Index table above
5. Test inline in a DAX measure in Power BI Desktop before committing

---

## Related References

- **colors.md** — Semantic color tokens and hex values
- **typography.md** — SVG font rules (system fonts only)
- **components.md** — KPI card patterns that use trend icons
- **wordmark/SKILL.md** — Wordmark SVG assets (light and dark)

---

**Version:** 2.0
**Last Updated:** 2026-05-02
