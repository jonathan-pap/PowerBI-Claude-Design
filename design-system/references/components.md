# Component Library Reference

## Overview

The component library defines the design specifications for all reusable UI elements in Power BI reports. Each component includes dimensions, colors, typography, and Power BI implementation details.

All components follow the design system's color palette (blue primary, semantic colors for status), typography scale (Segoe UI, 10pt–28pt), and 8px grid alignment.

---

## KPI Card

The KPI card is a foundational component used in executive dashboards, header zones, and detail pages. It displays a single metric with context.

### Dimensions

- **Width:** 188–596px (depending on layout; common: 233px for 5-card row)
- **Height:** 88–120px (88px in KPI zone; 120px if standalone)
- **Padding:** 12px all sides
- **Border radius:** 4px (subtle rounding)

### Structure

```
┌─────────────────────────┐
│  Label (11pt gray)      │  ← Top: metadata
├─────────────────────────┤
│  1,245,890              │  ← Large number (16pt bold)
│                         │
│  ↑ +5.2%  (green text) │  ← Trend (icon + percent)
└─────────────────────────┘
```

### Content Details

| Element | Font | Size | Color | Notes |
|---------|------|------|-------|-------|
| **Label** | Segoe UI | 11pt regular | #605E5C (gray) | Describes metric. Single line. |
| **Value** | Segoe UI | 16pt bold | #0078D4 (blue) or semantic | Large number. Right-aligned optional. |
| **Trend** | Segoe UI | 11pt regular | #107C10 (green) or #D13438 (red) | Icon (↑↓) + percentage. Green if positive, red if negative. |

### Styling

- **Background:** #FFFFFF (white)
- **Border:** 1px solid #E7E6E6 (light gray)
- **Padding:** 12px
- **Shadow:** Optional: 2px blur, 4px spread, rgba(0,0,0,0.08)
- **Hover state:** Background lightens to #F3F2F1; border darkens to #D2D0CE

### Variants

#### Variant A: Metric + Trend (Standard)
```
Total Revenue
1,245,890
↑ +5.2%
```

#### Variant B: Metric Only (No Trend)
```
Total Customers
18,450
```

#### Variant C: Metric + Status Badge
```
Pipeline Value
$4.2M
🟢 On Track
```

#### Variant D: Metric + Comparison
```
This Month
$890K
vs. $850K Last Month (+4.7%)
```

### Configuration

Visual type: `card`

| Property | Value |
|----------|-------|
| Width | 233px (5-card row) or 296px (4-card row) |
| Height | 88px |
| Background color | #FFFFFF |
| Border color | #E7E6E6, 1px |
| Value font | Segoe UI, 16pt, bold, #0078D4 |
| Label font | Segoe UI, 11pt, regular, #605E5C |
| Padding | 12px all sides |

Bind the **Values** bucket to the target measure. Set the card title to the metric name (11pt Segoe UI, #605E5C).

### Accessibility

- Label clearly identifies the metric (no mystery numbers)
- Color (green/red) paired with icon (↑↓) for colorblind accessibility
- Contrast ratio 7:1 minimum (met by dark numbers on white background)
- 88×88px minimum touch target (met by standard card dimensions)

---

## Slicer (Filter Control)

Slicers allow users to filter data by category, date, or dimension. Two main styles are defined: dropdown and button.

### Slicer: Dropdown Style

**Best for:** Many categories (20+), compact space, primary filter

**Dimensions:**
- **Width:** 150–250px (common: 200px)
- **Height:** 32px
- **Padding:** 8px left, 8px right

**Content:**
- Closed state: "Region ▼" (label + caret)
- Open state: dropdown menu with checkbox options

**Styling:**
- **Background (inactive):** #FFFFFF (white)
- **Border:** 1px solid #D2D0CE (gray)
- **Text:** 12pt regular, #201F1E (dark)
- **Caret:** Primary blue (#0078D4)
- **Hover:** Background #F3F2F1, border #0078D4
- **Open/Active:** Border #0078D4 (2px solid)

**Example:**
```
┌─ Region ────────────▼─┐
│ ☑ All (12)           │
│ ☑ Northeast (4)      │
│ ☐ Southeast (3)      │
│ ☐ Midwest (5)        │
└──────────────────────┘
```

### Slicer: Button Style

**Best for:** Few categories (2–6), high visibility, prominent filtering

**Dimensions:**
- **Width:** 100–150px per button
- **Height:** 40px
- **Spacing between:** 8px
- **Padding:** 8px all sides

**Content:**
- Button label (category name)
- Active button highlighted

**Styling:**
- **Background (inactive):** #E7E6E6 (light gray)
- **Text (inactive):** 12pt regular, #201F1E
- **Background (active):** #0078D4 (primary blue)
- **Text (active):** 12pt regular, #FFFFFF (white)
- **Hover (inactive):** Background #D2D0CE
- **Border:** None (flat design)

**Example:**
```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Northeast   │  │   MIDWEST   │  │ Southeast   │
│ (inactive)  │  │  (active)   │  │ (inactive)  │
└─────────────┘  └─────────────┘  └─────────────┘
```

### Multi-Select Behavior

Both styles support multiple selections:
- **Dropdown:** Checkboxes allow multiple checked items
- **Buttons:** Hold Ctrl/Cmd to multi-select (or single-select mode if preferred)
- **Clear:** "Clear All" button or "All" option to reset

### Configuration

Visual type: `slicer`

#### Dropdown Slicer

| Property | Value |
|----------|-------|
| Width | 200px |
| Height | 32px |
| Style | Dropdown |
| Field bucket | Single column (e.g. `Geography[Region]`) |
| Background | #FFFFFF |
| Border | 1px #D2D0CE; active/selected: 2px #0078D4 |
| Text | 12pt regular, #201F1E |

#### Button Slicer

| Property | Value |
|----------|-------|
| Width | 100–150px per button |
| Height | 40px |
| Style | Tile / Button |
| Inactive background | #E7E6E6, text #201F1E |
| Active background | #0078D4, text #FFFFFF |
| Multi-select | Off for year/period selectors |

---

## Matrix / Table

Tables and matrices display detailed data with rows and columns. Used for drill-down, transaction lists, and summary tables.

### Dimensions

- **Width:** 500–1248px (full-width or half-width)
- **Height:** 160–400px (flexible based on row count)
- **Row Height:** 36px (including padding)
- **Column Width:** 80–200px (depending on data)
- **Padding:** 8px cell padding

### Column Header

- **Background:** Primary blue (#0078D4)
- **Text:** 14pt semibold, white (#FFFFFF)
- **Padding:** 8px
- **Border-bottom:** 2px #0078D4

### Data Rows

- **Background (normal):** #FFFFFF
- **Background (alternate):** #EFF6FC (light blue, every other row)
- **Text:** 12pt regular, #201F1E
- **Padding:** 8px
- **Border-bottom:** 1px #E7E6E6 (subtle divider)

### Conditional Formatting

Common patterns:

#### Pattern 1: Value-Based Background
```
If value > threshold: Green background
If value < threshold: Red background
Otherwise: White background
```

#### Pattern 2: Text Color Based on Metric
```
If sales trend positive: Green text
If sales trend negative: Red text
Otherwise: Dark gray text
```

#### Pattern 3: Data Bar (In-Cell)
```
│▓▓▓▓▓░░░░│ 65%
```

### Interaction

- **Row hover:** Background lightens to #F3F2F1
- **Sort:** Click column header to sort ascending/descending
- **Drill:** Click value to drill to detail page (if configured)
- **Expand:** Click row expand arrow (if hierarchical)

### Configuration

Visual type: `matrix` (or `tableEx` for flat tables)

| Property | Value |
|----------|-------|
| Width | 500–1248px |
| Height | 160–400px |
| Row height | 36px |
| Column header background | #0078D4 |
| Column header text | 14pt semibold, #FFFFFF |
| Row background (normal) | #FFFFFF |
| Row background (alternate) | #EFF6FC |
| Row text | 12pt regular, #201F1E |
| Cell padding | 8px |
| Row border | 1px #E7E6E6 |

Bind **Rows** and **Columns** buckets to dimension columns; bind **Values** to measures.

**Conditional formatting** — configure via Format pane > Conditional formatting per value column:
- `>= 1,000,000` → background #107C10 (green)
- `< 500,000` → background #D13438 (red)
- Otherwise → default (white)

---

## Line Chart

Line charts show trends over time. Used for time-series analysis, performance tracking, and forecasts.

### Line Specification

- **Line weight:** 2.5px (visible but not heavy)
- **Line color:** Use 8-color categorical palette (one color per series)
- **Line style:** Solid (not dashed, unless representing forecast)
- **Marker (point):** 5px circle, same color as line
- **Marker visibility:** Show on all data points or hide if many points

### Axes

- **X-axis (category):** Time periods (dates, months, etc.)
  - Label font: 10pt regular, #605E5C
  - Tick spacing: 4–8 week intervals for annual data
  - Format: "MMM YYYY" (Jan 2026)

- **Y-axis (value):** Numeric scale
  - Label font: 10pt regular, #605E5C
  - Grid lines: Subtle gray (#F3F2F1) at low opacity
  - Format: Currency ($), percentage (%), or count as appropriate

### Data Labels

- **Visibility:** Optional (off by default; enable for key points)
- **Font:** 10pt regular, #201F1E
- **Position:** Above line (not overlapping)
- **Format:** Rounded to 2 decimal places

### Legend

- **Position:** Below chart or right side
- **Font:** 11pt regular
- **Color swatch:** Match line color exactly
- **Click to toggle:** Show/hide series on click

### Gridlines

- **Horizontal:** Yes, subtle (#F3F2F1) at 0.3 opacity
- **Vertical:** No (too much visual noise)
- **Zero line:** Bold (#605E5C) if scale includes negative values

### Configuration

Visual type: `lineChart`

| Property | Value |
|----------|-------|
| Width | 600px |
| Height | 300px |
| Line weight | 2.5px |
| Line colors | 8-color categorical palette (in order) |
| X-axis font | 10pt regular, #605E5C |
| Y-axis font | 10pt regular, #605E5C |
| Gridlines | Horizontal only, color #F3F2F1, opacity 30% |
| Legend position | Bottom |
| Data labels | Off by default |

Bind the **Axis** bucket to a date/time column (e.g. `Time[Month]`), **Legend** to a category column, and **Values** to the measure(s).

---

## Bar & Column Chart

Bar charts (horizontal) and column charts (vertical) compare values across categories.

### Bar Specification

- **Bar width:** 60% of available space (40% gap between bars)
- **Bar color:** Single color (primary blue #0078D4) or categorical palette (if series)
- **Bar spacing:** Equal gaps, aligned to grid
- **Rounded corners:** Optional; if present, 2px radius

### Data Labels

- **Font:** 12pt regular, #201F1E
- **Position:** Outside bars (right for horizontal, top for vertical)
- **Format:** Rounded integer or currency
- **Visibility:** On by default for value clarity

### Axes

- **Category axis (X for column, Y for bar):**
  - Font: 10pt regular, #605E5C
  - Labels: Category names, left-aligned
  
- **Value axis (Y for column, X for bar):**
  - Font: 10pt regular, #605E5C
  - Grid lines: Horizontal (subtle #F3F2F1)
  - Format: Currency, percentage, or count

### Interaction

- **Hover:** Bar highlights with darker shade of same color
- **Click (if drill):** Navigate to detail page
- **Drill-up button:** If hierarchical

### Stacked Variants

For stacked bars/columns:
- **Stacked bar:** Segments within same bar, one color per segment
- **100% stacked:** Normalized to percentage scale
- **Legend:** Identify all segments

### Configuration

Visual type: `columnChart` (vertical) or `barChart` (horizontal)

| Property | Value |
|----------|-------|
| Width | 600px |
| Height | 300px |
| Bar width | ~60% of available column width |
| Single-series color | #0078D4 (primary blue) |
| Multi-series colors | 8-color palette in order |
| Data labels | On, outside bars, 12pt regular, #201F1E |
| Axis labels | 10pt regular, #605E5C |
| Gridlines | Horizontal, subtle #F3F2F1 |

For a single-series chart, bind **Axis** to a category column and **Values** to a measure. For stacked/grouped, also bind **Legend** to a category column. Sort by value descending (unless time-based).

---

## Navigation Buttons

Navigation buttons provide drill-through, page navigation, and action triggers.

### Dimensions

- **Width:** 100–150px
- **Height:** 40px
- **Padding:** 12px horizontal, 8px vertical
- **Spacing (adjacent buttons):** 8px

### Styling

| State | Background | Border | Text | Font |
|-------|------------|--------|------|------|
| **Inactive/Default** | #E7E6E6 | 1px #D2D0CE | #201F1E | 12pt semibold |
| **Hover** | #D2D0CE | 1px #A19F9D | #201F1E | 12pt semibold |
| **Active/Selected** | #0078D4 | 1px #0078D4 | #FFFFFF | 12pt semibold |
| **Disabled** | #E7E6E6 | 1px #E7E6E6 | #A19F9D | 12pt regular (not bold) |

### Icon Buttons

Optional icon within button:
- **Icon size:** 16×16px
- **Padding (icon + text):** 8px between
- **Position:** Left-aligned (icon before text)

**Examples:**
- "← Back" (chevron-left icon)
- "⟳ Refresh" (refresh icon)
- "→ Next" (chevron-right icon)

### Button Group

Multiple buttons aligned horizontally:
- **Spacing:** 8px between buttons
- **Common set:** ["Back", "Next", "Done"] or ["Yes", "No", "Cancel"]
- **Primary action:** Highlighted in primary blue
- **Secondary actions:** Gray background

### Configuration

Use the **Button** visual type in Power BI Desktop (Insert > Button).

| State | Fill color | Border | Text color | Font |
|-------|-----------|--------|-----------|------|
| Secondary/default | #E7E6E6 | 1px #D2D0CE | #201F1E | 12pt semibold |
| Primary action | #0078D4 | 1px #0078D4 | #FFFFFF | 12pt semibold |
| Hover | #D2D0CE | 1px #A19F9D | #201F1E | 12pt semibold |
| Disabled | #E7E6E6 | 1px #E7E6E6 | #A19F9D | 12pt regular |

Width: 120px, Height: 40px. Set the **Action** to Page navigation, Bookmark, or URL as needed. For page nav buttons, set the **Destination** to the target page in the Action pane.

---

## Page Header

The page header establishes branding, title, and primary navigation context.

### Header Structure

- **Height:** 60px (standard) or 80px (with logo)
- **Background:** Primary blue (#0078D4) or light gray (#F3F2F1)
- **Padding:** 12px all sides
- **Content:** Logo (optional) + Title + Slicers (optional)

### Content Alignment

```
┌────────────────────────────────────────────────────────┐
│  [Logo] Sales Dashboard          [Region ▼] [Year ▼]  │
│         28pt bold title          Slicers (right)       │
└────────────────────────────────────────────────────────┘
```

### Title (28pt Bold)

- **Font:** Segoe UI semibold or bold
- **Color:** White (#FFFFFF) on blue; dark (#201F1E) on light
- **Alignment:** Left, 16px from edge
- **Truncation:** Ellipsis if too long (max 400px)

### Logo (if present)

- **Size:** 32×32px or 40×40px
- **Margin:** 8px right of title
- **Alignment:** Vertical center

### Slicers in Header (optional)

- **Position:** Right side, 16px from edge
- **Font:** 12pt
- **Spacing:** 8px between slicers
- **Max width:** 500px (leave ~300px buffer for title)

### Breadcrumb (Drill-Through Pages)

If on a detail/drill page:
- **Position:** Below title, 12pt regular
- **Format:** "Dashboard > Product Category > SKU"
- **Color:** Gray (#605E5C)
- **Click behavior:** Navigate to parent page

### Configuration

| Element | Type | x | y | width | height | Notes |
|---------|------|---|---|-------|--------|-------|
| Background | Shape (rectangle) | 0 | 0 | 1280 | 60 | Fill #0078D4, no border |
| Title | Text box | 16 | 12 | 400 | 36 | 28pt bold, #FFFFFF, left-aligned |
| Slicer 1 | Slicer | 900 | 14 | 180 | 32 | Dropdown, right-aligned |
| Slicer 2 | Slicer | 1096 | 14 | 180 | 32 | Dropdown, right-aligned |

For a light header: fill #F3F2F1, title text #201F1E 20pt semibold.

---

## Tooltip Page (Hover Details)

Tooltip pages are small overlays that appear on hover, providing detail without navigation.

### Dimensions

- **Width:** 320px
- **Height:** 200px (flexible based on content)
- **Position:** Centered on hovered data point or right side of cursor

### Content Structure

```
┌──────────────────────────┐
│ Product: Widget Pro       │  ← Category/Name
│ Sales: $45,000            │  ← Primary metric
│ Growth: ↑ 12.5%           │  ← Trend
│                           │
│ vs. Previous: $40,000     │  ← Comparison
│ Target: $50,000           │  ← Goal/Benchmark
└──────────────────────────┘
```

### Styling

- **Background:** #FFFFFF (white)
- **Border:** 1px solid #D2D0CE
- **Shadow:** 8px blur, 4px spread, rgba(0,0,0,0.12)
- **Padding:** 16px all sides
- **Border-radius:** 4px

### Content

- **Title:** 12pt semibold, #201F1E
- **Value:** 14pt semibold, #0078D4 or semantic color
- **Label:** 10pt regular, #605E5C
- **Divider:** 1px #E7E6E6 between sections

### Interaction

- **Trigger:** Hover over data point
- **Auto-hide:** Disappears on mouse leave or after 10 seconds
- **Click behavior:** Optional; can drill on click

### Configuration

Tooltip formatting is set per visual in the **Format pane > Tooltips**:

| Property | Value |
|----------|-------|
| Tooltip type | Default or Report page |
| Font size | 12pt |
| Background color | #FFFFFF |
| Border color | #D2D0CE |

To create a custom tooltip page in Power BI:
1. Create a new page with `type: "tooltip"` (320×240px)
2. Add visuals showing detail fields
3. Link the tooltip page to source visual via drill-through or interaction

---

## Summary & Checklist

When implementing components:

- [ ] Use the specified dimensions (width, height, padding)
- [ ] Apply correct colors from the primary and semantic palettes
- [ ] Use Segoe UI for all text (no fallback fonts needed in Power BI)
- [ ] Match font sizes exactly (12pt body, 16pt titles, etc.)
- [ ] Maintain 8px grid alignment for layout
- [ ] Provide semantic color for status (green positive, red negative, amber warning)
- [ ] Ensure minimum 44×44px touch targets for interactive elements
- [ ] Test contrast ratios (7:1 for primary text, 4.5:1 minimum for labels)
- [ ] Include data labels or legends for clarity
- [ ] Document any custom variants for team reference

---

**Reference Document**  
**Version:** 1.0  
**Last Updated:** 2026-04-30
