# Layout System Reference

## Canvas Dimensions

### Standard Power BI Report Page

- **Width:** 1280px
- **Height:** 720px
- **Aspect Ratio:** 16:9 (landscape, optimized for monitor displays)
- **Safe Area:** 1248×704px (16px margins on all sides)
- **Max Visual:** 1248px wide × 704px tall (within safe area)

**Why 1280×720?**  
This is the standard Power BI report canvas size, optimized for 1080p displays and typical monitor viewports. It balances readability with the ability to fit multiple visuals without scrolling.

### Alternate Dimensions

For special use cases:

| Use Case | Width | Height | Aspect | Notes |
|----------|-------|--------|--------|-------|
| Standard Page | 1280px | 720px | 16:9 | Most common. Desktop dashboard. |
| Mobile View | 640px | 1080px | 3:5 | Vertical. Responsive layout. |
| Tablet View | 1024px | 768px | 4:3 | Landscape. iPad-sized. |
| Widescreen | 1920px | 1080px | 16:9 | Extended display. Rare in Power BI. |

For this design system, we standardize on **1280×720px** unless specified otherwise.

---

## Grid System

The grid system provides a consistent framework for aligning visual elements.

### Base Unit

- **Grid Unit:** 8px
- **Half Unit:** 4px
- **Double Unit:** 16px (primary gutter width)

All spacing and sizing should align to multiples of 8px where possible:
- Margins: 16px, 24px, 32px
- Padding: 8px, 12px, 16px, 20px, 24px
- Gaps between visuals: 8px, 16px

### Grid Layout (Columns & Rows)

The 1280px width divides evenly into a **8-column grid**:

- **Column Width:** 140px per column (1280 ÷ 8)
- **Gutter:** 16px between columns
- **Total Content Width:** 1248px (within safe margin)

### Example Grid Divisions

| Layout | Columns | Width Each | Usage |
|--------|---------|-----------|-------|
| Full width | 1 | 1248px | Single large chart, full-width table |
| Half-width | 2 | 596px | Two charts side-by-side |
| Third-width | 3 | 380px | Three charts in a row |
| Quarter-width | 4 | 268px | Four small cards or KPIs in a row |
| Sixths | 6 | 188px | Six KPI cards in a row |

Use the grid math above to compute exact `x`, `y`, `width`, `height` values for each visual. Example positions for a 3-row, 2-column layout with 16px margin and 16px gutter:

| Visual | x | y | width | height |
|--------|---|---|-------|--------|
| Column chart (top-left) | 16 | 16 | 612 | 160 |
| Card (top-right) | 644 | 16 | 620 | 160 |
| Bar chart (full-width) | 16 | 192 | 1248 | 160 |
| Matrix (full-width) | 16 | 368 | 1248 | 160 |

---

## Safe Margins

All visuals respect safe margins to prevent content from appearing cut off or too close to edges.

### Standard Safe Margins

- **Left margin:** 16px (1 gutter unit)
- **Right margin:** 16px
- **Top margin:** 16px (inside page, below header if present)
- **Bottom margin:** 16px

**Total safe area:** 1248px wide × 704px tall (within 1280×720 canvas)

### No Safe Margin Areas

- Header zone (if full-width header extends edge-to-edge)
- Page background (can extend edge-to-edge)

---

## Five Standard Layout Templates

Each template provides a tested arrangement pattern. Choose the template that best fits your reporting goal.

### Template 1: KPI Row + Chart Grid

**Best for:** Executive summaries, sales dashboards, performance overviews

**Structure:**
- Header zone (60px): title + optional slicers
- KPI zone (120px): 4-6 KPI cards in a single row
- Chart zone (remainder): 2-column grid of charts and tables

**Visual Hierarchy:**
- Emphasizes top-level metrics (KPIs)
- Supporting detail charts below
- Clear scanning flow: metrics → drill-down data

**Layout Code:**

```
Layout:
- Row 0 (Header): Title + Slicers [1248×60]
- Row 1 (KPIs): Card | Card | Card | Card [6-card row, 120px]
- Row 2 (Charts): ColumnChart [596px] | BarChart [596px]
- Row 3 (Charts): Matrix [1248px, spanning full width]
```

**Recommended For:** P&L dashboards, revenue dashboards, operations centers

### Template 2: Left Nav + Main Canvas

**Best for:** Drill-through pages, detail exploration, multi-section reports

**Structure:**
- Left navigation (250px): buttons or links to different sections
- Main canvas (1030px): single large visual, slicer controls, or multi-step drill
- Header zone (60px): title, back button, breadcrumb

**Visual Hierarchy:**
- Navigation on left for persistent context
- Main area focuses on current selection
- Clear drill-through path

**Layout Code:**

```
Layout:
- Header [top]: Title + Back Button [1248×60]
- Left Nav [left]: 8 Buttons, 40px tall each [250×320]
- Main [right]: Large Chart or Slicer [980×560]
```

**Recommended For:** Drill-through detail pages, product category explorers, customer deep-dives

### Template 3: Full-Width Analytics

**Best for:** Technical analysts, deep data exploration, comprehensive metrics

**Structure:**
- Minimal header (40px): just title, no major slicers
- Full-width visuals stacked vertically
- Multiple tables and charts, no KPI card zone

**Visual Hierarchy:**
- Maximum vertical real estate for charts
- Maximizes data density
- Suitable for power users

**Layout Code:**

```
Layout:
- Header [top]: Title only [1248×40]
- Chart 1: Full-width Line Chart [1248×180]
- Chart 2: Full-width Matrix [1248×240]
- Chart 3: Full-width Scatter [1248×160]
```

**Recommended For:** Analyst dashboards, SQL performance reports, ETL monitoring

### Template 4: Executive Summary

**Best for:** C-suite dashboards, board presentations, high-level overviews

**Structure:**
- Large header (80px): branding, large title, date/time
- Headline KPI (200px): single large number + trend
- Three-column metric cards (100px): supporting KPIs
- Single large chart (remainder): primary insight visual

**Visual Hierarchy:**
- Dominant headline metric
- Supporting context cards
- Single primary chart
- Minimal noise, maximum clarity

**Layout Code:**

```
Layout:
- Header [top]: Logo + Title + Date [1248×80]
- Headline KPI: Large Card [1248×200]
- Supporting: Card | Card | Card [3-card row, 100px]
- Chart: Full-width LineChart [1248×240]
```

**Recommended For:** Executive presentations, board reviews, investor dashboards

### Template 5: Detail Drill-Through

**Best for:** Transaction-level data, support/operations, detailed inquiries

**Structure:**
- Compact header (40px): minimal chrome
- Detail table (full height): large matrix with many columns
- Optional right sidebar (300px): related metrics, drill options
- Tooltip or detail page: drill-through to even finer detail

**Visual Hierarchy:**
- Table as primary focus
- Supporting metrics on right (optional)
- Dense information architecture

**Layout Code:**

```
Layout:
- Header [top]: Back Button + Breadcrumb [1248×40]
- Main: Large Matrix [948×680, left side]
- Sidebar [right]: KPI Stack [300×680]
```

**Recommended For:** Support tickets, transaction ledgers, error logs, order details

---

## Header Zone (60px)

The header zone anchors the page and provides navigation/filtering context.

### Header Structure

- **Height:** 60px
- **Background:** Primary blue (#0078D4) or light gray (#F3F2F1)
- **Content:** Title (left) + Slicers (right) or just title
- **Padding:** 16px left, 16px right, 12px top, 12px bottom

### Header Components

#### Title

- **Font:** 28pt bold or 20pt semibold (depending on header bg)
- **Color:** White (#FFFFFF) on blue; dark (#201F1E) on light gray
- **Alignment:** Left-aligned
- **Width:** ~400px minimum

#### Slicers

- **Count:** 0–3 slicers in header
- **Style:** Dropdown or button
- **Alignment:** Right-aligned
- **Spacing:** 16px between slicers
- **Height:** 32px (fits comfortably in 60px header with padding)

#### Back Button (Drill-Through Pages Only)

- **Position:** Left side, before title
- **Icon:** Chevron left or back arrow
- **Function:** Navigate to parent page
- **Click Target:** 44×44px minimum

### Header Visual Positions

| Element | Type | x | y | width | height | Notes |
|---------|------|---|---|-------|--------|-------|
| Background shape | rectangle | 0 | 0 | 1280 | 60 | Fill: #0078D4 |
| Title text box | textBox | 16 | 12 | 400 | 36 | 28pt bold, white |
| Slicer 1 | slicer | 850 | 14 | 200 | 32 | Dropdown style |
| Slicer 2 | slicer | 1066 | 14 | 200 | 32 | Dropdown style |

---

## KPI Card Zone (120px)

The KPI zone displays key performance indicators and sits directly below the header.

### KPI Zone Structure

- **Height:** 120px
- **Rows:** 1 (single row of cards)
- **Columns:** 4–6 cards depending on metrics
- **Card Spacing:** 16px between cards
- **Padding:** 16px top, 16px bottom, 16px left, 16px right

### Card Dimensions

| Card Count | Width Each | Calculation |
|------------|-----------|------------|
| 4 cards | 296px | (1248 - 3×16) ÷ 4 |
| 5 cards | 233px | (1248 - 4×16) ÷ 5 |
| 6 cards | 188px | (1248 - 5×16) ÷ 6 |

Each card is 88px tall (120px zone - 32px padding).

### KPI Card Content

- **Large number:** 16pt bold, primary color
- **Label:** 11pt regular, secondary color
- **Trend indicator:** Icon (↑ green, ↓ red) + % change
- **Background:** White with subtle border

---

## Chart Zone (Remainder)

The chart zone occupies all space below the KPI zone.

### Chart Zone Space Calculation

```
Available Height = 720px (total) - 60px (header) - 120px (KPI) - 32px (margins)
                 = 508px
```

Use this space for:
- 2 large charts (254px each, with 16px gutter between)
- 3 stacked charts (160px each, with 16px gutters)
- 1 full-height chart (508px)
- Mixed: 1 chart (300px) + matrix (190px) + metrics sidebar (300px)

### Typical Arrangements

**2-Column Grid:**
```
Chart 1 [Left]  [16px]  Chart 2 [Right]
596px                   596px
```

**3-Row Stack:**
```
Chart 1: 160px
[16px gutter]
Chart 2: 160px
[16px gutter]
Chart 3: 160px
```

**Full-Width + Half-Width:**
```
Chart 1 (full): 1248px  [108px height]
[16px gutter]
Chart 2 (left): 596px   [200px height]
[16px gutter]
Chart 3 (right): 596px  [200px height]
```

---

## Spacing & Gutters

Consistent spacing ensures alignment and visual harmony.

### Vertical Spacing

| Context | Spacing | Usage |
|---------|---------|-------|
| Zone to zone (header to KPI, KPI to charts) | 16px | Major section breaks |
| Visual to visual (charts on same row/column) | 16px | Standard gutter between visuals |
| Within visual (title to data) | 8px | Internal padding |
| Paragraph or line break | 12px | Within text content |

### Horizontal Spacing

| Context | Spacing | Usage |
|---------|---------|-------|
| Page edge to content | 16px | Safe margin |
| Column to column (grid) | 16px | Standard gutter |
| Element to element (buttons, slicers) | 8-12px | Tight grouping |
| Slicer to slicer (header) | 16px | Header spacing |

---

## Responsive Considerations

While Power BI primarily targets desktop (1280×720), consider these principles:

### Mobile Layout Strategy (if needed)

- Stack all columns into single column
- Full-width KPI cards (one per row if tall)
- Hide or collapse slicers into dropdown
- Drill-through pages work better than side-by-side layout

### Density Adjustments

- **High Density:** Reduce margins to 8px, use 6-column KPI cards
- **Low Density:** Increase margins to 24px, use 4-column KPI cards, larger chart areas
- **Medium Density:** Standard 16px margins, 5-card KPI row (recommended default)

---

## Layout Validation Checklist

Before finalizing a layout:

- [ ] All visuals align to 8px grid
- [ ] 16px margins on all page edges
- [ ] 16px gutters between visuals
- [ ] Header height is 60px (or 40px for minimal)
- [ ] KPI zone is 120px (if present)
- [ ] Page uses one of five template patterns (or documented variant)
- [ ] No visual overlaps
- [ ] All visuals within safe area (1248×704px)
- [ ] Title and slicers fit in header without truncation
- [ ] No text extends beyond visual boundaries

---

**Reference Document**  
**Version:** 1.0  
**Last Updated:** 2026-04-30
