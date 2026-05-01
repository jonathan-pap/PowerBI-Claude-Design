---
name: design-system
description: Power BI design system for building consistent, accessible, professional reports. Covers the 3-30-300 Rule (visual hierarchy), detail gradient layouts, KPI design principles, anti-patterns ("Power BI Slop"), typography, color palettes, spacing, themes, and evaluation checklists. Emphasizes intentional pre-attentive attributes, semantic colors, and minimizing cognitive load.
version: 2.0
---

# Power BI Design System

## Overview

The Power BI Design System provides a comprehensive set of standards and patterns for building professional, accessible, and consistent data visualizations and reports. This system applies Microsoft's Fluent design principles adapted for analytics and business intelligence contexts, ensuring reports answer specific questions, minimize cognitive load, and guide attention intentionally rather than relying on decoration.

### When to Use This Design System

Consult this design system when:

- **Building new reports or dashboards** — reference layouts, component styles, and the 3-30-300 Rule
- **Planning visual hierarchy** — understand the detail gradient (KPIs → Charts → Tables)
- **Designing KPI cards** — every KPI must answer two questions: "Is this good or bad?" and "Is it getting better or worse?"
- **Generating theme JSON** — apply color palettes and visual styles programmatically
- **Formatting visuals** — data label sizing, axis typography, gridline subtlety, pre-attentive attributes
- **Creating themed pages** — header zones, KPI cards, chart arrangements, spacing
- **Ensuring accessibility** — color contrast, semantic color usage, readable font sizes, alt text
- **Evaluating reports** — use the report evaluation checklist to audit design quality
- **Avoiding "Power BI Slop"** — generic, poorly formatted, template-looking reports that lack clarity

### Design System Files

This knowledge base consists of reference documents:

- **colors.md** — Primary palette, semantic colors, data visualization colors, theme JSON mappings
- **typography.md** — Font family, type scale, usage rules, Power BI property mappings
- **layouts.md** — Canvas dimensions, grid system, 5 standard layout templates, safe margins
- **components.md** — KPI cards, slicers, matrices, charts, navigation, page headers, tooltips

Consult the specific reference file based on your task. All files include Power BI Format pane property mappings and theme JSON so you can translate design specs into exact configuration steps.

---

## Core Design Principles

### 1. Consistency

All reports follow the same visual language:
- **Color palette** is applied consistently across all data visualizations
- **Typography scale** ensures hierarchy and readability
- **Component spacing** uses an 8px grid system for alignment
- **Layout structure** follows one of five validated templates
- **Theme usage** applies semantic meaning (green=good, red=bad, neutral=gray/yellow)

Consistency builds trust and reduces cognitive load—users should feel they are using one coherent product, not a collection of disparate reports.

### 2. Accessibility

Design for all users:
- **Color blindness safe** — data visualization palette is WCAG AAA compliant for colorblind users; blues instead of greens with reds
- **Sufficient contrast** — primary text (white on dark blue) meets 7:1 WCAG AA standard; 4.5:1 minimum for body text
- **Readable typography** — body text at 12pt minimum, display text at 28pt for emphasis
- **Semantic colors** — green for positive, red for negative, amber for warnings—consistent meaning across reports
- **Alt text on all visuals** — screen reader friendly descriptions of charts and KPIs
- **Keyboard navigation** — slicer buttons and drill-through buttons are keyboard accessible
- **Minimal animations** — avoid drop shadows and animations that can trigger vestibular issues
- **Don't rely on color alone** — pair semantic colors with icons, labels, or secondary cues

### 3. Hierarchy

Visual hierarchy guides attention and understanding:
- **3-30-300 Rule** — most important content (KPIs) top-left; least important/most detailed content (tables, drill-down) bottom-right
- **Detail gradient** — arrange content vertically: Summary (cards) → Analysis (charts) → Details (tables)
- **Type scale** creates clear distinction between headings, body text, and captions
- **Color weight** (saturation and value) draws focus to KPIs and key insights
- **Spatial hierarchy** — headers at top, KPIs below, detailed charts below, drill-through access nested
- **Emphasis through size** — 28pt display text for dashboard titles, 12pt for data labels, 10pt for axis labels
- **Pre-attentive attributes** — styles and colors steer attention to what matters; formatting is intentional, not aesthetic

### 4. Answer Specific Questions

Avoid "Power BI Slop" (generic, poorly formatted, template-looking reports):
- Every report must answer specific business questions
- Every visual must earn its space — remove visuals that don't contribute to answering a question
- Minimize cognitive load — simplify, don't add unnecessary decoration
- Focus on clarity over impressiveness

---

## Detail Gradient Layout

Arrange content vertically by level of detail:

```
+------------------+------------------+
|   KPIs/Cards     |   KPIs/Cards     |  ← Top: High-level, important
|   (Summary)      |   (Summary)      |     Numbers only; max 4-6 KPIs
+------------------+------------------+
|                                     |
|   Charts/Trends                     |  ← Middle: Context, trends
|   (Analysis)                        |     Provides context; shows patterns
|                                     |
+------------------+------------------+
|                                     |
|   Tables/Details                    |  ← Bottom: Detailed data
|   (Drill-down)                      |     Row-by-row; sortable; detailed metrics
|                                     |
+------------------+------------------+
```

**Reading order:** Most viewers scan top-to-bottom, left-to-right. Place the answer at the top. Details below.

---

## Core Rules

1. **3-30-300 Rule** — Most important/least detailed content top-left. Least important/most detailed content bottom-right. Viewers absorb the summary and drill down only if they need details.

2. **All pages need a title** — 28pt Segoe UI Semibold, positioned at x=24, y=24, width=400-600px, height=48-64px. Titles provide context for screen readers.

3. **Equal spacing between visuals, equal margins** — Calculate positions arithmetically. Min 16px gap between visuals. 24-32px margins on all sides. No hand-placed offsets.

4. **Use a custom theme** — Don't rely on Power BI defaults. Custom themes ensure consistency and control.

5. **Reports depend on the semantic model** — Most functionality comes from DAX, relationships, and model design. Keep the report layer thin.

6. **Thin report measures/visual calculations** — Use sparingly; only for report-specific scenarios that don't belong in the model.

7. **All data visuals must have field bindings** — Fields must exist in the semantic model. No ad-hoc calculations that could be DAX.

8. **Smart chart selection** — Visual vocabulary is essential. Choose the right chart type for the data. Sort by value descending unless time-based.

9. **Pre-attentive attributes** — Styles and colors steer attention, not decorate. Formatting is intentional. Store in theme, not bespoke visual config.

10. **Colors: muted and soft** — Semantic colors (red=bad/green=good) only when encoding meaning. Consider colorblindness — blues instead of greens with reds. Never use bright/saturated colors purely for aesthetics.

11. **Fonts: Segoe UI and Segoe UI Semibold** — Don't use custom fonts. Check readability at report size. Limited font sizes only.

---

## Visual Count Limits

Enforce these limits to maintain clarity and performance:

| Metric | Limit | Reason |
|--------|-------|--------|
| Visuals per page | 12-15 max | Performance; cognitive load |
| KPI/Card visuals at top | 4-6 max | Summary zone clarity |
| Slicer visuals per page | 3 max | Use filter pane for additional filters |
| Pages per report | 5-8 max | Navigation; discoverability |

---

## Quick Reference

### Primary Color Palette

| Role | Color | Hex | Usage |
|------|-------|-----|-------|
| Primary Blue | Blue 600 | #0078D4 | Interactive elements, primary CTA buttons, active states |
| Secondary Blue | Blue 500 | #106EBE | Hover states, secondary emphasis |
| Tertiary Blue | Blue 400 | #3A96DD | Disabled states, subtle backgrounds |
| Light Blue | Blue 100 | #DEECF9 | Card backgrounds, light emphasis |
| White | — | #FFFFFF | Page background, card backgrounds, text on dark |

### Semantic Colors

- **Good (Positive/Success):** #107C10 (green) — growth, increase, achievement
- **Bad (Negative/Error):** #D13438 (red) — decline, loss, problems
- **Neutral (Info):** #605E5C (gray) — informational, default state
- **Warning (Caution):** #FFB900 (amber) — alerts, warnings, pending

### Typography

- **Font Family:** Segoe UI (Power BI default); Segoe UI Semibold for emphasis
- **Page Title:** 28pt, semibold
- **Visual Titles:** 16pt, semibold
- **Body Text:** 12pt, regular weight
- **Data Labels:** 12pt, regular
- **Axis Labels:** 10pt, regular

### Grid & Spacing

- **Base Unit:** 8px
- **Canvas:** 1280×720px (default) or 1920×1080px
- **Gutter (gap between visuals):** 16px minimum
- **Edge Margin:** 24-32px all sides
- **Page Title Position:** x=24, y=24, width=400-600px, height=48-64px
- **Minimum Touch Target:** 44×44px

### Visual Spacing Rules

- Minimum gap between visuals: 16px
- Standard edge margins: 24-32px
- Equal spacing is mandatory — calculate all positions arithmetically, not by eye
- Alignment guides: use grid templates from **layouts.md**

---

## When to Apply Design Standards

### Report Creation

When building a new report:
1. Choose a layout template from **layouts.md** (KPI row + grid, executive summary, detail drill-through, etc.)
2. Apply the detail gradient (KPIs → Charts → Tables)
3. Select primary data visualization colors from the 8-color categorical palette in **colors.md**
4. Apply typography scale from **typography.md** — 16pt for visual titles, 12pt for data labels
5. Use component specifications from **components.md** for KPI cards, slicers, and page headers

### Theme JSON Generation

When applying consistent theming programmatically:
1. **Check theme wildcards first** — Review `visualStyles["*"]["*"]` in the theme before overriding individual visual properties
2. **Use ThemeDataColor over hex** — Prefer `{"ThemeDataColor": {"ColorId": 1, "Percent": 0}}` over hardcoded hex values to allow flexibility
3. **Apply semantic colors consistently** — Use "good" (green), "bad" (red), "neutral" (gray/yellow) throughout the report
4. **Only override in visual.json if truly one-off** — Most formatting should live in the theme, not bespoke per-visual config
5. **Test color contrast** — Use the accessibility checklist (7:1 ratio for normal text, 4.5:1 for large text)

### KPI Card Design

Every KPI must answer two questions to have meaning:

1. **"Is this good or bad?"** — Requires a target + gap. Display the gap with conditional formatting (green if favorable, red if unfavorable).
2. **"Is it getting better or worse?"** — Requires a trend. Use a trend indicator (arrow up/down, sparkline, or secondary metric).

**Design rules:**
- Prefer KPI visual type over Card when a target exists
- Apply conditional formatting to the gap, not the primary value
- Pair color with a secondary cue (arrow, icon, or label) — never rely on color alone
- Round aggressively at summary level ("518M" not "517,893,412")
- Use the "20% change test" — if this metric changed 20%, should someone act differently? If not, it's not a KPI.

**Structure:**
- Primary value (large, 28-32pt)
- Target or previous period (smaller, 12pt)
- Gap or trend (color-coded, 12pt, with icon or arrow)

### Visual Formatting

When formatting individual visuals:
1. **Titles:** 16pt Segoe UI semibold (see **typography.md** for the exact Format pane settings)
2. **Data labels:** 12pt regular weight
3. **Axis labels:** 10pt regular weight, gray text (#605E5C)
4. **Gridlines:** subtle gray (#E7E6E6) at low opacity; thinner than default
5. **Data point colors:** use 8-color palette; if more than 8 series, apply sequential hue instead
6. **Legend/labels:** identify all colors without relying on hue alone; pair with icons if encoding meaning

### Table Design: "Subtract, Don't Add"

Remove visual noise to let data speak:
- **Remove gridlines** — especially heavy borders between rows
- **Remove heavy row banding** — let whitespace naturally separate rows
- **Sort by most important measure** — often variance or performance delta
- **Apply data bars** to primary measure only
- **Apply color scales** to variance columns only, not raw values
- **Disable unnecessary axis labels** and legend text
- **Lighter gridlines** — subtle gray at low opacity

### Layout & Spacing

When arranging visuals on a page:
1. **Establish title zone** at top (48-64px) with 28pt page title at x=24, y=24
2. **KPI card zone** (120px) directly below title for key metrics (max 4-6 cards)
3. **Chart zone** (remaining canvas) divided by grid system (see **layouts.md**)
4. **Apply 24-32px safe margins** on all sides
5. **Use 16px minimum gutters** between visual containers
6. **Calculate all positions arithmetically** — no hand-placed offsets; use grid layout

### Accessibility: Alt Text Pattern

Add alt text to all visuals for screen reader users:

```json
"altText": {
  "expr": {
    "Literal": {
      "Value": "'Line chart showing monthly sales trend from Jan to Dec, peaking in Q4'"
    }
  }
}
```

Pattern: `"[Chart type] showing [what data] [key insight or context]"`

---

## Design Tokens (Reference)

| Token | Value | Usage |
|-------|-------|-------|
| `--color-primary` | #0078D4 | Primary interactive elements |
| `--color-good` | #107C10 | Success, uptrend, positive delta |
| `--color-bad` | #D13438 | Error, downtrend, negative delta |
| `--color-neutral` | #605E5C | Informational, default state |
| `--color-warning` | #FFB900 | Caution, warning, pending |
| `--color-text-primary` | #201F1E | Primary body text |
| `--color-text-secondary` | #605E5C | Secondary, subdued text |
| `--font-body` | 12pt Segoe UI | Standard text |
| `--font-title` | 16pt Segoe UI SemiBold | Visual titles |
| `--font-display` | 28pt Segoe UI SemiBold | Page titles |
| `--spacing-unit` | 8px | Grid base unit |
| `--spacing-gutter` | 16px | Min space between visuals |
| `--spacing-margin` | 24-32px | Page edge margin |

---

## Accessibility Checklist

Before publishing a report:

- [ ] All text is at least 12pt and high contrast (7:1 ratio for normal text, 4.5:1 for large text)
- [ ] Data visualization uses the 8-color colorblind-safe palette
- [ ] Semantic colors (green/red/amber) are paired with icons or labels (don't rely on color alone)
- [ ] Slicer buttons and navigation have at least 44×44px click targets
- [ ] Page headers include a title visible to screen readers (alt text)
- [ ] All visuals have alt text describing content and key insights
- [ ] KPI cards show both numeric value and trend indicator (arrow, icon, or secondary metric)
- [ ] Chart gridlines are subtle gray, not black or high-contrast
- [ ] Legend or data labels identify all colors without relying on hue alone
- [ ] No unnecessary drop shadows or animations (vestibular accessibility)
- [ ] Sufficient whitespace; minimal cognitive load
- [ ] Page title positioned clearly at x=24, y=24 with readable font size

---

## Report Evaluation Checklist

Use this checklist to audit a completed report for design quality:

1. **Page count** — 5-8 max? (Navigation, discoverability)
2. **Visuals per page** — 12-15 max? (Performance, clarity)
3. **Theme usage** — Custom theme applied? Wildcards used before visual-level overrides?
4. **Layout & Spacing** — Equal spacing arithmetically calculated? Title present? Detail gradient (KPIs → Charts → Tables)? Less than 3 slicers?
5. **KPI Design** — Every KPI answers "good/bad?" and "getting better/worse?" (target + trend)?
6. **Color Consistency** — Conditional formatting sparse? Colors muted? Semantic colors only encoding meaning?
7. **Info:Ink Ratio** — Lighter gridlines? Disabled unnecessary axes/labels? Whitespace effective?
8. **Readability** — Font sizes sufficient at report size? Body text 12pt+? Titles 16pt+?
9. **Accessibility** — Alt text on all visuals? Minimal shadows? Sufficient contrast (7:1)?
10. **Font Consistency** — Limited font sizes? Only Segoe UI and Segoe UI Semibold?
11. **Sorting** — Charts sorted by value descending (unless time-based)? Tables sorted by most important measure?
12. **Purpose Clarity** — Does the report answer specific questions? No "Power BI Slop" (generic, poorly formatted, template-looking)?

---

## Common Pitfalls to Avoid

### Power BI Slop

Generic, poorly formatted, template-looking reports that:
- Don't answer specific questions
- Overuse decoration (bright colors, heavy shadows, unnecessary fonts)
- Pack too many visuals without clear hierarchy
- Rely on color alone without supporting context
- Have inconsistent spacing and alignment

**Fix:** Focus on clarity. Every visual must earn its space. Answer specific questions. Use the detail gradient. Pair color with secondary cues (icons, labels, trends).

### Bare KPI Numbers

A KPI showing only "518M" without context lacks meaning.

**Fix:** Add target + gap AND trend. Example:
- Primary: 518M (sales)
- Secondary: Target 550M (gray)
- Gap: -32M (red, down arrow)

### Color Overload

Bright, saturated colors for every category; red/green for all positive/negative semantics.

**Fix:** Use muted colors. Semantic colors only encode meaning. Consider colorblindness (blues instead of greens with reds). Pair color with icons or labels.

### Inconsistent Spacing

Visuals hand-placed with no alignment grid.

**Fix:** Calculate all positions arithmetically using the grid system in **layouts.md**. 16px minimum gap between visuals. 24–32px margins from page edges. Equal spacing is mandatory.

### Missing Alt Text

Visuals with no screen reader descriptions.

**Fix:** Add alt text to all visuals: `"[Chart type] showing [data] [key insight]"`

### Overuse of Slicers

More than 3 slicers cluttering the page.

**Fix:** Use filter pane for additional filters. Max 3 slicers per page, positioned neatly in the title zone.

---

## When to Push Back on Requests

If a user request contradicts these guidelines, explain better alternatives:

- **Request:** "Make this report look impressive and pretty"
  - **Response:** "Let's focus on clarity instead. What specific questions should this report answer? I'll design the layout to guide viewers to the answers."

- **Request:** "Add all these metrics as KPIs"
  - **Response:** "KPIs need targets + trends to have meaning. Let's identify which metrics are truly business-critical, then design them properly with targets and trends."

- **Request:** "Use more colors to make it visually interesting"
  - **Response:** "Muted colors and semantic meaning work better. Let's pair colors with icons/labels and ensure readers understand what the colors mean."

- **Request:** "Fit 30 visuals on this page"
  - **Response:** "Too many visuals hurt performance and clarity. Let's follow the detail gradient—KPIs at top, charts in middle, details at bottom—and limit to 12-15 total."

---

## Related References

- **colors.md** — Full color palette, semantic colors, accessibility contrast ratios, and theme JSON
- **typography.md** — Type scale, font weight rules, usage by component, Power BI property names
- **layouts.md** — Canvas dimensions, grid system, five layout templates, spacing specifications
- **components.md** — Component specs for KPI cards, slicers, matrices, charts, buttons, headers, tooltips

### Applying Changes in Power BI Desktop

- **Theme:** Save the design system JSON as a `.json` file, then apply via **View > Themes > Browse for themes**
- **Visual formatting:** Use the **Format pane** (paint roller icon) to set font sizes, colors, borders, and data labels per visual
- **Conditional formatting:** Configure per column via the **Format pane > Conditional formatting** for KPI gaps and status indicators
- **Layout:** Set visual positions using the **x**, `y`, `width`, `height` properties in the PBIP report JSON files, or drag and align visuals in Power BI Desktop using snap-to-grid

---

**Version:** 2.0  
**Last Updated:** 2026-04-30  
**Maintainer:** Design System Team
