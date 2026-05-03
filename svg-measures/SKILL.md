---
name: svg-measures
description: Reference guide for generating SVG markup via DAX measures in Power BI. Covers KPI icons, sparklines, bullet charts, gauges, status pills, jitter plots, waterfalls, and IBCS bars rendered as data:image/svg+xml strings with dataCategory ImageUrl. Includes TMDL declaration patterns, axis normalization, HASONEVALUE guards, and the <desc> sort trick.
version: 2.0
---

# SVG Measures for Power BI

Generate inline SVG graphics using DAX measures that return SVG markup strings. These render as images in table, matrix, card, image, and slicer visuals when the measure's `dataCategory` is set to `ImageUrl`.

---

## How It Works

1. A DAX measure returns an SVG string prefixed with `data:image/svg+xml;utf8,`
2. The measure's `dataCategory` is set to `ImageUrl` (in TMDL or model tooling)
3. Power BI renders the SVG as an image in the visual's image slot

### Supported Visuals

| Visual | Notes | Reference |
|--------|-------|-----------|
| Table (`tableEx`) | `grid.imageHeight` / `grid.imageWidth` | `references/svg-table-matrix.md` |
| Matrix (`pivotTable`) | Same as table | `references/svg-table-matrix.md` |
| Image (`image`) | `sourceType='imageData'` + `sourceField` | `references/svg-image-visual.md` |
| Card (New) (`cardVisual`) | `callout.imageFX` | `references/svg-card-slicer.md` |
| Slicer (New) (`advancedSlicerVisual`) | Header image slot | `references/svg-card-slicer.md` |

> **Classic card (`card`) does NOT support SVG.** Only the new `cardVisual` works.

---

## DAX Measure Structure — The VAR Pattern

Every SVG measure must follow a strict VAR-based structure. Organize into four clearly labelled regions:

```dax
SVG Measure Name =

-- CONFIG: input measures, scope column, visual parameters
VAR _Actual        = [Sales Amount]
VAR _Target        = [Sales Target]
VAR _Scope         = ALLSELECTED( 'Product'[Category] )
VAR _Font          = "Segoe UI"
VAR _FontSize      = 10

-- CONFIG: colors
VAR _ActualColor   = "#5B8DBE"
VAR _TargetColor   = "#333333"

-- NORMALIZATION: scale raw values to SVG coordinate space
VAR _BarMin        = 0
VAR _BarMax        = 100
VAR _MaxInScope    = CALCULATE( MAXX( _Scope, [Sales Amount] ), REMOVEFILTERS( 'Product'[Category] ) )
VAR _AxisMax       = _MaxInScope * 1.1
VAR _AxisRange     = _BarMax - _BarMin
VAR _ActualNorm    = DIVIDE( _Actual, _AxisMax ) * _AxisRange

-- SVG ELEMENTS: one VAR per visual element (back to front = assembly order)
VAR _SvgPrefix     = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 25'>"
VAR _Sort          = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
VAR _Bar           = "<rect x='0' y='5' width='" & _ActualNorm & "' height='15' fill='" & _ActualColor & "'/>"
VAR _TargetLine    = "<rect x='" & DIVIDE( _Target, _AxisMax ) * _AxisRange & "' y='2' width='2' height='21' fill='" & _TargetColor & "'/>"
VAR _SvgSuffix     = "</svg>"

-- ASSEMBLY: concatenate in render order (first = back layer, last = front)
VAR _SVG           = _SvgPrefix & _Sort & _Bar & _TargetLine & _SvgSuffix

RETURN _SVG
```

**Rules:**
- CONFIG section first — the only section users ever need to change
- NORMALIZATION section — always scale raw values before using as coordinates
- One VAR per element — never concatenate elements inline in a single expression
- ASSEMBLY at the end — document order = z-order (first renders behind)

---

## Axis Normalization

Raw measure values (e.g. 1,234,567) cannot be used as pixel coordinates. Scale all values to a fixed SVG range:

```dax
-- 1. Define the SVG coordinate range
VAR _BarMin = 0         -- leftmost position (or offset for labels)
VAR _BarMax = 100       -- rightmost position

-- 2. Find the maximum across all rows visible in the visual
VAR _Scope      = ALLSELECTED( 'Table'[GroupColumn] )
VAR _MaxInScope = CALCULATE( MAXX( _Scope, [Measure] ), REMOVEFILTERS( 'Table'[GroupColumn] ) )
VAR _AxisMax    = _MaxInScope * 1.1    -- 10% headroom so bars never clip the edge

-- 3. Normalize each value
VAR _AxisRange  = _BarMax - _BarMin
VAR _Normalized = DIVIDE( _Actual, _AxisMax ) * _AxisRange
```

- Use `ALLSELECTED` when the chart should respond to slicers
- Use `ALL` for a fixed axis regardless of filter context
- For charts with two series (actual + target), take the max of both:
  ```dax
  VAR _AxisMax =
      IF(
          HASONEVALUE( 'Table'[GroupColumn] ),
          MAX( _MaxActual, _MaxTarget ),
          CALCULATE( MAX( [Actual], [Target] ), REMOVEFILTERS( 'Table'[GroupColumn] ) )
      ) * 1.1
  ```

---

## HASONEVALUE Guard

Table and matrix SVG measures must return `BLANK()` on subtotal/total rows:

```dax
RETURN
    IF(
        HASONEVALUE( 'Table'[GroupColumn] ),
        _SvgPrefix & _Sort & _Bar & _SvgSuffix,
        BLANK()
    )
```

Without this guard, the measure evaluates on grand total rows with meaningless aggregated coordinates.

---

## The `<desc>` Sort Trick

Embed a zero-padded value in a `<desc>` tag so Power BI can sort the image column by the underlying number:

```dax
VAR _Sort = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
```

Include `_Sort` immediately after `_SvgPrefix` in assembly. Power BI uses the `<desc>` content as the sort key when the user clicks the SVG column header.

---

## CONCATENATEX for Time-Series Data

Build point strings for sparklines and multi-point charts:

```dax
VAR _Points = CONCATENATEX(
    _SparklineTable,
    [@X] & "," & (100 - [@Y]),   -- Y is inverted: SVG Y=0 is at top
    " ",
    [Date], ASC
)
-- Produces: "0,80 10,60 20,40 30,20"
-- Use in: <polyline points='...'/>
```

Note the Y inversion: `(viewBoxHeight - [@Y])` flips values so higher numbers render higher on screen.

---

## SVG Rules in Power BI

### Quoting
- Use **single quotes** for all SVG XML attributes — avoids DAX double-quote escaping
- DAX double quotes inside strings: escape as `""` (standard DAX)

### Colors
- **Always use hex with `#` directly** — e.g., `fill='#2196F3'`
- **Never use `%23`** — URL encoding causes `VisualDataProxyExecutionUnknownError` in image visuals
- **Never use named colors** (`blue`, `red`) — unreliable in Power BI's SVG renderer
- For transparency: use 8-digit hex — e.g., `fill='#FFFFFF00'` (fully transparent)

### Fonts
- **System fonts only**: `Segoe UI`, `Arial`, `Calibri`, `sans-serif`
- **No external fonts** — `@font-face` and Google Fonts URLs do not load in Power BI's SVG sandbox
- Recommended: `font-family='Segoe UI, Arial, sans-serif'`

### External Resources
- **No external URLs** — images, fonts, and `href` links to external domains are blocked
- **No JavaScript** — `<script>` tags are stripped; SVG must be purely declarative
- **No CSS animations** — `<style>` blocks may be ignored

### Size Limits
- **32K character limit** on the rendered SVG string (not the DAX expression)
- `CONCATENATEX` over 30+ rows easily hits this — prefer `<polyline>` over individual `<circle>` dots
- Integer coordinates over decimals save characters: `ROUND( _x, 0 )` → `INT( _x )`
- Split complex designs into two simpler measures rather than one enormous one

### Coordinate System
- Y=0 is at the **top** — invert chart values: `_ViewBoxHeight - _NormalizedValue`
- Use `viewBox` with a 0–100 range for normalized coordinates
- Elements render in document order (first = back, last = front)

---

## Number Formatting

Adaptive scale labels for any magnitude:

```dax
VAR _Label = SWITCH( TRUE(),
    _Actual <= 1E3,  FORMAT( _Actual, "#,0" ),
    _Actual <= 1E6,  FORMAT( _Actual, "#,0, K" ),
    _Actual <= 1E9,  FORMAT( _Actual, "#,0,, M" ),
    _Actual <= 1E12, FORMAT( _Actual, "#,0,,, bn" )
)
```

---

## TMDL Declaration

### Measure with `dataCategory: ImageUrl`

```tmdl
createOrReplace

	measure 'Sales Sparkline' =
			VAR _Values =
					ADDCOLUMNS(
							CALCULATETABLE( VALUES( 'Date'[Month] ), DATESINPERIOD( 'Date'[Date], MAX( 'Date'[Date] ), -12, MONTH ) ),
							"@Value", [Sales Amount]
					)
			VAR _XMin   = MIN( 'Date'[Month] )
			VAR _XMax   = MAX( 'Date'[Month] )
			VAR _YMin   = MINX( _Values, [@Value] )
			VAR _YMax   = MAXX( _Values, [@Value] )
			VAR _Points =
					CONCATENATEX(
							ADDCOLUMNS( _Values, "@X", INT( 100 * DIVIDE( 'Date'[Month] - _XMin, _XMax - _XMin ) ), "@Y", INT( 30 * DIVIDE( [@Value] - _YMin, _YMax - _YMin ) ) ),
							[@X] & "," & ( 30 - [@Y] ),
							" ",
							'Date'[Month]
					)
			VAR _Prefix = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 30'>"
			VAR _Line   = "<polyline fill='none' stroke='#5B8DBE' stroke-width='2' points='" & _Points & "'/>"
			VAR _Suffix = "</svg>"
			RETURN
					IF( HASONEVALUE( 'Product'[Category] ), _Prefix & _Line & _Suffix, BLANK() )
		dataCategory: ImageUrl
		formatString: ""
		displayFolder: "SVG Charts"
		description: "Line sparkline — last 12 months of sales per category row"
```

**TMDL properties for SVG measures:**

| Property | Value | Required |
|----------|-------|----------|
| `dataCategory` | `ImageUrl` | **Yes** — tells Power BI to render as image |
| `formatString` | `""` | Yes — leave empty for text/URL measures |
| `displayFolder` | `"SVG Charts"` | Recommended — keeps SVG measures grouped |
| `description` | freeform | Recommended |

---

## When to Use SVG Measures

| Scenario | Best Choice |
|----------|-------------|
| Inline micro-charts in table/matrix rows | **SVG measure** |
| KPI indicators in card visuals | **SVG measure** |
| Lightweight status pills, progress bars | **SVG measure** |
| Complex charts with cross-filtering or tooltips | **Deneb** |
| Statistical charts (distributions, regressions) | **Python/R visual** |
| Large interactive dashboards | **Deneb or certified custom visual** |

SVG measures shine for **simple, non-interactive inline graphics** that don't need click handlers or complex data transforms.

---

## Community Libraries

Before writing custom SVG DAX, check whether an existing library already provides the chart type:

| Library | Author | Key Chart Types |
|---------|--------|-----------------|
| DaxLib.SVG | Jake Duddy | Area, line, boxplot, heatmap, jitter, violin, bar |
| PBI-Core-Visuals-SVG-HTML | David Bacci | Chips, tornado, gradient matrix |
| PowerBI MacGuyver Toolbox | Stepan Resl / Data Goblins | 20+ bar, 14+ line, 24+ KPI templates |
| Dashboard-Design UDF Library | Dashboard-Design | Target-line bars, pill visuals |
| Kerry Kolosko Templates | Kerry Kolosko | Sparklines, data bars, KPI cards |
| PowerofBI.IBCS | Andrzej Leszkiewicz | IBCS bar, waterfall, pin, small multiples |

Libraries are installed into the **semantic model** (not the report) as DAX UDFs. See `dax-udf/SKILL.md` for installation patterns.

---

## Reference Files

Pattern libraries by visual type — each file contains ready-to-adapt DAX patterns:

- **`references/svg-elements.md`** — SVG element syntax reference: rect, circle, line, polyline, text, path, group, gradient
- **`references/svg-table-matrix.md`** — Table/Matrix patterns: data bar, bullet chart, overlapping bars, dumbbell, lollipop, status pill, sparklines (line, area, bar)
- **`references/svg-image-visual.md`** — Image visual patterns: KPI header card, sparkline with endpoint, multi-metric dashboard tile
- **`references/svg-card-slicer.md`** — Card (`cardVisual`) and Slicer patterns: arrow indicator, mini gauge, mini donut, progress bar

---

## Example Files

Ready-to-use DAX expressions in `examples/`:

| File | Chart Type | Target |
|------|-----------|--------|
| `sparkline-measure.dax` | Line sparkline (polyline + CONCATENATEX) | Table/Matrix |
| `progress-bar-measure.dax` | Conditional progress bar | Table/Matrix/Card |
| `dumbbell-chart-measure.dax` | Actual vs target dumbbell | Table/Matrix |
| `bullet-chart-measure.dax` | Bullet chart with sentiment action dots | Table/Matrix |
| `overlapping-bars-measure.dax` | Overlapping bars with variance label | Table/Matrix |
| `overlapping-bars-with-variance-measure.dax` | Overlapping bars + variance bar + icon | Table/Matrix |
| `lollipop-conditional-measure.dax` | Lollipop with scaled dot + label | Table/Matrix |
| `ibcs-bar-measure.dax` | IBCS horizontal bar (AC vs PY + delta) | Table/Matrix |
| `boxplot-measure.dax` | Box-and-whisker (Q1/Median/Q3 + whiskers) | Table/Matrix |
| `jitter-plot-measure.dax` | Dot strip chart with pseudo-random jitter | Table/Matrix |
| `waterfall-measure.dax` | Waterfall with OFFSET cumulative positioning | Table/Matrix |
| `status-pill-measure.dax` | Rounded pill badge with category color | Table/Matrix |
| `target-bar-measure.dax` | Horizontal track with coloured zones + diamond needle | Image/Card |
| `cache-hit-ratio-measure.dax` | Same pattern — Cache Hit Ratio production example | Image/Card |

All examples use `__PLACEHOLDER` tokens for measure names and column references. Replace these before deploying.

---

## Related References

- **tmdl-standards/SKILL.md** — TMDL syntax ground rules: measure declaration, `createOrReplace` wrapper, indentation
- **measures/SKILL.md** — Scalar DAX measures that SVG measures wrap and visualise
- **calc-groups/SKILL.md** — Calc group context affects which measure value is selected; SVG measures must handle `SELECTEDMEASURE()` output
- **dax-udf/SKILL.md** — UDFs for shared SVG string utilities (color pickers, path builders, label formatters)

---

**Version:** 2.0  
**Last Updated:** 2026-05-02
