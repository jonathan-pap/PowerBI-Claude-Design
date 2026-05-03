# Typography Reference

## Font Family

### Primary Font: Segoe UI

Segoe UI is the default font in Power BI and aligns with Microsoft's Fluent design system. It is designed for screen readability and comes installed on all Windows systems.

- **Font Name:** Segoe UI
- **Fallback:** Arial
- **Style:** Regular (400), Semibold (600), Bold (700)
- **Supported:** All platforms (Windows, Mac, Web)

**Why Segoe UI?**
- Designed for digital screens with open letterforms and generous spacing
- Clear distinction between similar characters (1, l, I)
- Professional appearance suitable for business intelligence
- Consistent rendering across Power BI Desktop and Web

### Font Weights

| Weight | Value | Usage |
|--------|-------|-------|
| Regular | 400 | Body text, data labels, axis labels, default text |
| Semibold | 600 | Visual titles, card headers, emphasis |
| Bold | 700 | Page titles, key highlights, strong emphasis |

**Note:** Power BI rendering sometimes shows "Semi Bold" (with space) in UI. Both "Semibold" and "Semi Bold" reference weight 600.

---

## Type Scale

The type scale establishes a clear visual hierarchy from page titles down to tiny captions. Each size serves a specific purpose.

### Scale Definition

| Role | Size | Weight | Line Height | Usage |
|------|------|--------|-------------|-------|
| **Display** | 28pt | Bold | 36px | Page/dashboard title, primary heading |
| **Heading 1** | 20pt | Semibold | 28px | Section title, top-level category |
| **Heading 2** | 16pt | Semibold | 24px | Visual title, card title, subsection |
| **Heading 3** | 14pt | Semibold | 20px | Minor heading, slicer label |
| **Body** | 12pt | Regular | 18px | Data label, axis label, body text |
| **Small** | 11pt | Regular | 16px | Secondary label, helper text, metadata |
| **Caption** | 10pt | Regular | 14px | Tiny label, footnote, timestamp, axis tick |

### Visual Representation

```
Display:     Lorem Ipsum
             28pt Bold

Heading 1:   Lorem Ipsum
             20pt Semibold

Heading 2:   Lorem Ipsum
             16pt Semibold

Heading 3:   Lorem Ipsum
             14pt Semibold

Body:        Lorem Ipsum
             12pt Regular

Small:       Lorem Ipsum
             11pt Regular

Caption:     Lorem Ipsum
             10pt Regular
```

---

## Usage Rules by Component

### Page Title (Display, 28pt Bold)

- Located at the top of the page, typically in the header zone
- Single line or small multiple lines
- Communicates the primary topic of the entire report
- Example: "Sales Performance Dashboard Q1 2026"

**Format pane setting:** Visual title → Font: Segoe UI, Size: 28pt, Weight: Bold

### Visual Title (Heading 2, 16pt Semibold)

- Appears above each chart, table, or card
- Describes what the visual shows
- Example: "Revenue by Region" above a bar chart

**Format pane setting:** General → Title → Font: Segoe UI, Size: 16pt, Weight: Semibold

### KPI Card Value (Body, 12pt → 16pt Bold)

- Primary metric value on a KPI card
- Typically bold to emphasize the number
- Size depends on card dimensions; 16pt bold is standard for 120px tall cards
- Example: "1,245,890" (revenue total)

**Format pane setting:** Callout value → Font: Segoe UI, Size: 16pt, Bold, Color: #0078D4

### KPI Card Label (Small, 11pt Regular)

- Descriptive label below or beside the value
- Identifies what the KPI represents
- Example: "Total Revenue" below "1,245,890"

**Format pane setting:** Category label → Font: Segoe UI, Size: 11pt, Regular, Color: #605E5C

### Data Labels (Body, 12pt Regular)

- Text displayed on or near chart bars, columns, or points
- Should be readable without overwhelming the visual
- Example: "15%" on a stacked bar chart segment

**Format pane setting:** Data labels → Font: Segoe UI, Size: 12pt, Regular

### Axis Labels (Caption to Small, 10-11pt Regular)

- X-axis labels: category names
- Y-axis labels: value ranges
- Should be small to avoid clutter but still readable
- Example: "Jan 2026", "Feb 2026" on X-axis; "0", "500K", "1M" on Y-axis

**Format pane setting:** X-axis / Y-axis → Font: Segoe UI, Size: 10pt, Regular, Color: #605E5C

### Tooltip Text (Body, 12pt Regular)

- Hover tooltips over data points
- Provides additional detail without cluttering the visual
- Default font size is usually acceptable; ensure contrast is high

**Format pane setting:** Tooltips → Font size: 12pt

### Matrix/Table Header (Heading 3, 14pt Semibold)

- Column headers in tables and matrices
- Bold to distinguish from data rows
- Example: "Region", "Sales", "Growth %"

**Format pane setting:** Column headers → Font: Segoe UI, Size: 14pt, Bold, Color: #FFFFFF, Background: #0078D4

Note: In Power BI, the column header formatting section is labeled **Column headers** in matrices and **Column headers** / **Header** in table visuals.

### Matrix/Table Data (Body, 12pt Regular)

- Standard data rows
- Consistent with axis labels and body text
- Example: "Northeast", "2,450,000", "12.5%"

**Format pane setting:** Values → Font: Segoe UI, Size: 12pt, Regular, Color: #201F1E

### Slicer Labels (Small, 11pt Regular)

- Labels identifying slicer controls
- Example: "Region", "Time Period", "Product Category"

**Format pane setting:** Values → Font: Segoe UI, Size: 11pt, Regular

### Legend (Small, 11pt Regular)

- Identifies series or categories in a chart
- Should be small but readable
- Example: "Sales", "Cost", "Profit" in a stacked bar chart

**Format pane setting:** Legend → Font: Segoe UI, Size: 11pt, Regular

---

## Font Weight Rules

### When to Use Semibold (600)

- Visual titles (Heading 2, 16pt)
- Card titles (Heading 3, 14pt)
- Section headers
- Key emphasis text
- First occurrence of important terms

**Semibold commands attention without being as heavy as bold.**

### When to Use Regular (400)

- Body text (12pt and smaller)
- Data labels
- Axis labels
- Helper text
- Secondary information
- Tooltip content

**Regular weight is default for most text.**

### When to Use Bold (700)

- Page titles (Display, 28pt)
- Primary metric values in KPI cards
- Critical alerts or warnings
- Key numbers that deserve emphasis

**Reserve bold for the most important information. Overuse reduces its impact.**

### Anti-Pattern: Never Use All Caps

Power BI follows Fluent conventions—avoid setting text to ALL CAPS for emphasis. Use font size and weight instead. All caps:
- Reduces readability
- Takes up more visual space
- Can feel aggressive in business contexts

✗ "TOTAL SALES"  
✓ "Total Sales" (with larger font or semibold weight)

---

## Power BI Format Pane Reference

These are the Format pane sections to look for when applying typography settings in Power BI Desktop. Select a visual, open the Format pane (paint roller), and navigate to the section listed below.

### Common Format Pane Sections

| Section | Properties | Applied to |
|---------|-----------|------------|
| General → Title | Font, Size, Color, Bold | Visual titles above charts |
| Data labels | Font, Size, Color | Labels on bars, columns, points |
| X-axis / Category axis | Font, Size, Color | Horizontal axis labels |
| Y-axis / Value axis | Font, Size, Color | Vertical axis labels |
| Legend | Font, Size, Color | Series identification labels |
| Column headers | Font, Size, Color, Background | Table/matrix header row |
| Values | Font, Size, Color | Table/matrix data rows |
| Tooltips | Font size | Hover-over text |

### Visual Title

In the Format pane: **General > Title**
- Font family: Segoe UI
- Font size: 16pt (visual title) or 28pt (page title)
- Font weight: Semibold for charts, Bold for page titles
- Color: #201F1E (on light background)

### Data Labels

In the Format pane: **Data labels**
- Font family: Segoe UI
- Font size: 12pt
- Color: #201F1E

### Axes

In the Format pane: **X-axis** and **Y-axis**
- Font family: Segoe UI
- Font size: 10pt
- Color: #605E5C

---

## Text Color Pairing

All text should pair with appropriate foreground/background colors for contrast:

| Text Role | Color | Background | Contrast | Notes |
|-----------|-------|-----------|----------|-------|
| Primary body text | #201F1E | #FFFFFF | 13:1 | High contrast, very readable |
| Secondary text | #605E5C | #FFFFFF | 7:1 | Lower emphasis, still clear |
| Data labels (dark chart) | #201F1E | #FFFFFF | 13:1 | Default. High contrast. |
| Data labels (light chart) | #FFFFFF | #0078D4 | 7:1 | White text on colored background |
| Disabled text | #A19F9D | #FFFFFF | 4.5:1 | Minimum for AA compliance |
| Header text | #FFFFFF | #0078D4 | 7:1 | White on primary blue |

---

## Minimum Font Sizes

Respect these minimum font sizes for readability:

- **Body text & data labels:** 12pt minimum (11pt acceptable for secondary)
- **Axis labels:** 10pt minimum (9pt only for very high-density charts)
- **Captions & footnotes:** 10pt minimum
- **Disabled text:** Same size as enabled text; use color to indicate state, not size reduction

**Avoid:** Text smaller than 10pt except in rare cases (e.g., timestamp on a drill-through tooltip).

---

## Brand Fonts in SVG Measures

Power BI renders SVG measure output in a **sandboxed environment**. External font
loading — Google Fonts URLs, `@font-face` with CDN links, Adobe/Typekit kits —
**does not work**. Any web font reference silently falls back to the browser default,
which is why generated SVG looks wrong on screen.

**Only fonts already installed on the host machine are available.** On Windows
(Power BI Desktop and most Service environments), this means:

| Font | Status | Use for |
|------|--------|---------|
| Segoe UI | ✅ Always available | All text in SVG measures |
| Arial | ✅ Always available | Fallback only |
| Calibri | ✅ Windows / Office installs | Secondary fallback |
| sans-serif | ✅ Browser default | Final fallback |

### Correct `font-family` for SVG measures

```
font-family="Segoe UI, Arial, Calibri, sans-serif"
```

Use this string in every SVG `text` element and `style` block. Never reference
Google Fonts, Typekit, or any URL-based font in SVG measure DAX.

### What NOT to do

```xml
<!-- ✗ Will silently fall back — web fonts don't load in Power BI SVG -->
<style>@import url('https://fonts.googleapis.com/css?family=Inter');</style>
<text font-family="Inter, sans-serif">$1.2M</text>

<!-- ✓ Correct — system font, always available -->
<text font-family="Segoe UI, Arial, sans-serif" font-size="24">$1.2M</text>
```

### Font weight in SVG

SVG uses `font-weight` numeric values. Power BI maps these to Segoe UI variants:

| `font-weight` | Segoe UI variant | Use for |
|---------------|-----------------|---------|
| `400` | Regular | Body, labels |
| `600` | Semibold | Visual titles, card headers |
| `700` | Bold | KPI values, page titles |

---

## Font Pairing Strategy

Power BI uses a single-font system (Segoe UI) for consistency. Do not mix font families within a report.

- Primary: Segoe UI Regular, Semibold, Bold
- Fallback: Arial (if Segoe UI unavailable, though rare)
- Never use: Comic Sans, Georgia, or decorative fonts in dashboards

---

## Accessibility & Readability

### Line Length

- Ideal: 50–75 characters per line
- Maximum: 100 characters
- In Power BI: Avoid text boxes wider than 400px for body text

### Line Spacing (Leading)

- Display (28pt): 36px (1.3× font size)
- Heading (16-20pt): 24-28px (1.4–1.5× font size)
- Body (12pt): 18px (1.5× font size)
- Small text (10-11pt): 14-16px (1.4× font size)

Power BI defaults are generally adequate; adjust only if text appears cramped.

### Anti-Aliasing & Rendering

Segoe UI renders cleanly on both Windows and Mac. No special rendering settings needed.

---

**Reference Document**  
**Version:** 1.0  
**Last Updated:** 2026-04-30
