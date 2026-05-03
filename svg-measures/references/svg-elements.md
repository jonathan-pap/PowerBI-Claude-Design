# SVG Elements Reference for DAX

Quick reference for SVG elements used in DAX measures. All examples use single quotes for SVG attributes to avoid DAX double-quote escaping.

**Color rule:** always use hex with `#` directly — e.g., `fill='#2196F3'`. Never `%23`, never named colors.

---

## `<svg>` — Container

```dax
"<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 30'>"
```

| Attribute | Description |
|-----------|-------------|
| `xmlns` | **Required** on every `<svg>` element |
| `viewBox` | Coordinate system: `minX minY width height`. Use this instead of fixed `width`/`height` for responsive scaling |
| `width`, `height` | Optional fixed display size; viewBox takes precedence |
| `preserveAspectRatio` | `none` to stretch, `xMidYMid meet` to maintain ratio |
| `font-family` | Set once on root to inherit: `font-family='Segoe UI, Arial, sans-serif'` |

**Standard prefix pattern:**
```dax
VAR _SvgPrefix = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 25' font-family='Segoe UI, Arial, sans-serif'>"
VAR _SvgSuffix = "</svg>"
```

---

## `<rect>` — Rectangle

```dax
"<rect x='10' y='5' width='50' height='10' fill='#2196F3' rx='2'/>"
```

| Attribute | Description |
|-----------|-------------|
| `x`, `y` | Top-left position |
| `width`, `height` | Dimensions |
| `fill` | Fill color (`#hex`) |
| `stroke` | Border color |
| `stroke-width` | Border thickness |
| `opacity` | 0–1 transparency (whole element) |
| `fill-opacity` | Fill-only transparency |
| `rx`, `ry` | Corner radius (`rx='4'` for pill ends, `rx='50%'` for circle) |

**Common patterns:**
```dax
-- Background track
"<rect x='0' y='0' width='100' height='25' fill='#F5F5F5'/>"

-- Rounded progress bar
"<rect x='0' y='6' width='" & _W & "' height='13' fill='" & _Color & "' rx='6.5'/>"

-- Target line (thin rect)
"<rect x='" & _TargetX & "' y='2' width='2' height='80%' fill='#000000'/>"
```

---

## `<circle>` — Circle

```dax
"<circle cx='50' cy='12' r='5' fill='#F44336'/>"
```

| Attribute | Description |
|-----------|-------------|
| `cx`, `cy` | Center position |
| `r` | Radius |
| `fill`, `stroke`, `stroke-width`, `opacity` | Styling |

**Common patterns:**
```dax
-- Sentiment action dot (left edge)
"<circle cx='10' cy='12' r='5' fill='" & _SentimentColor & "'/>"

-- Lollipop endpoint (radius scales with value)
"<circle cx='" & (_BarMin + _ActualNorm) & "' cy='12' r='" & MAX( DIVIDE(_Actual, _AxisMax) * 7.5, 3.5 ) & "' fill='" & _Color & "'/>"

-- Dumbbell circle (outlined)
"<circle cx='" & _TargetNorm & "' cy='12' r='4' fill='#F5F5F5' stroke='#C7C7C7' stroke-width='1.5'/>"
```

---

## `<line>` — Straight Line

```dax
"<line x1='0' y1='12' x2='100' y2='12' stroke='#C7C7C7' stroke-width='1'/>"
```

| Attribute | Description |
|-----------|-------------|
| `x1`, `y1` | Start point |
| `x2`, `y2` | End point |
| `stroke` | Color |
| `stroke-width` | Thickness |
| `stroke-dasharray` | Dash pattern — e.g., `'4,2'` for dashed, `'2,2'` for dotted |
| `stroke-linecap` | `butt` (default), `round`, `square` |

**Common patterns:**
```dax
-- Axis baseline
"<line x1='0' y1='12' x2='100' y2='12' stroke='#C7C7C7'/>"

-- Dashed reference line
"<line x1='" & _AvgX & "' y1='1' x2='" & _AvgX & "' y2='23' stroke='#D64444' stroke-width='1.5' stroke-dasharray='3,2'/>"

-- Dumbbell connecting line
"<line x1='" & _ActualNorm & "' y1='12' x2='" & _TargetNorm & "' y2='12' stroke='" & _Fill & "' stroke-width='3'/>"
```

---

## `<polyline>` — Connected Points (Sparklines)

```dax
"<polyline fill='none' stroke='#01B8AA' stroke-width='2' points='0,25 20,15 40,20 60,5 80,10 100,8'/>"
```

| Attribute | Description |
|-----------|-------------|
| `points` | Space-separated `x,y` pairs |
| `fill` | `none` for line-only; color for filled area |
| `stroke` | Line color |
| `stroke-width` | Line thickness |
| `stroke-linejoin` | `round` for smooth corners |

**Build points with CONCATENATEX:**
```dax
VAR _Points = CONCATENATEX(
    ADDCOLUMNS( _Values,
        "@X", INT( 100 * DIVIDE( 'Date'[Month] - _XMin, _XMax - _XMin ) ),
        "@Y", INT( 30  * DIVIDE( [@Value]       - _YMin, _YMax - _YMin ) )
    ),
    [@X] & "," & ( 30 - [@Y] ),   -- invert Y: SVG Y=0 is top
    " ",
    'Date'[Month]
)
```

**Area sparkline** — close the shape by adding corner points:
```dax
"<polyline fill='url(#grad)' fill-opacity='0.3' stroke='#0078D4' stroke-width='2' points='0 30 " & _Points & " 100 30 Z'/>"
```

---

## `<text>` — Text Label

```dax
"<text x='50' y='16' font-size='10' fill='#333333' font-weight='600' text-anchor='middle' dominant-baseline='middle'>Label</text>"
```

| Attribute | Description |
|-----------|-------------|
| `x`, `y` | Anchor position (affected by `text-anchor` and `dominant-baseline`) |
| `font-size` | Size in SVG units |
| `fill` | Text color |
| `font-weight` | `normal`, `bold`, `600`, `700` |
| `font-family` | Use system fonts only: `Segoe UI, Arial, sans-serif` |
| `text-anchor` | Horizontal alignment: `start`, `middle`, `end` |
| `dominant-baseline` | Vertical alignment: `auto`, `middle`, `hanging` |

**Common patterns:**
```dax
-- Left-aligned value label
"<text x='" & _BarMin - 3 & "' y='15' font-size='10' font-weight='600' text-anchor='end' fill='" & _Color & "'>" & _Label & "</text>"

-- Centred text in pill
"<text x='50%' y='53%' font-size='10.5' font-weight='700' text-anchor='middle' dominant-baseline='middle' fill='" & _TextColor & "'>" & UPPER(_Status) & "</text>"

-- Tick axis label
"<text x='" & _X & "' y='52' font-size='11' fill='#888'>30</text>"
```

---

## `<path>` — Arcs, Curves, Polygons

```dax
"<path d='M 10,10 L 50,10 L 30,30 Z' fill='#4CAF50'/>"
```

| Command | Meaning | Example |
|---------|---------|---------|
| `M x,y` | Move to (start without drawing) | `M 10,10` |
| `L x,y` | Line to | `L 50,10` |
| `H x` | Horizontal line to x | `H 90` |
| `V y` | Vertical line to y | `V 50` |
| `A rx ry rot large-arc sweep x y` | Elliptical arc | `A 40 40 0 0 1 90 50` |
| `C x1 y1 x2 y2 x y` | Cubic Bézier | |
| `Q x1 y1 x y` | Quadratic Bézier | |
| `Z` | Close path | |

**Arc for gauge/donut (semi-circle):**
```dax
-- Background track (half circle, left to right)
"<path d='M 10 50 A 40 40 0 0 1 90 50' fill='none' stroke='#E0E0E0' stroke-width='8'/>"

-- Dynamic arc (progress gauge)
VAR _Rad      = ( _Pct * 180 - 90 ) * PI() / 180
VAR _EndX     = 50 + 40 * COS( _Rad )
VAR _EndY     = 50 + 40 * SIN( _Rad )
VAR _LargeArc = IF( _Pct > 0.5, 1, 0 )
-- "<path d='M 50 10 A 40 40 0 " & _LargeArc & " 1 " & _EndX & " " & _EndY & "' fill='none' stroke='#2196F3' stroke-width='8'/>"
```

**Up/Down arrow (triangle):**
```dax
-- Up arrow
"<polygon points='10,15 5,10 15,10' fill='#4CAF50'/>"

-- Down arrow
"<polygon points='10,5 5,10 15,10' fill='#F44336'/>"
```

**MDN path reference:** https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorials/SVG_from_scratch/Paths

---

## `<polygon>` — Closed Polygon

```dax
"<polygon points='50,14 61,35 86,35 68,57 75,80 50,65 25,80 32,57 14,35 39,35' fill='#FFD700'/>"
```

Useful for arrows, diamonds, stars. Same `points` format as `<polyline>` but automatically closes.

**Diamond needle head:**
```dax
"<polygon points='" & _NX & ",14 " & (_NX+5) & ",19 " & _NX & ",24 " & (_NX-5) & ",19' fill='#c97a3a'/>"
```

---

## `<g>` — Group

```dax
"<g transform='translate(10,5)'>" & _Element1 & _Element2 & "</g>"
```

| Transform | Example |
|-----------|---------|
| `translate(x, y)` | Shift all children |
| `rotate(angle)` | Rotate around origin |
| `scale(x, y)` | Scale children |

Groups can also inherit style attributes:
```dax
"<g fill='#888888' font-size='11'>" &
    "<text x='10' y='52'>0</text>" &
    "<text x='50' y='52'>50</text>" &
    "<text x='90' y='52'>100</text>" &
"</g>"
```

---

## `<defs>` + `<linearGradient>` — Gradient Fill

```dax
VAR _Defs =
    "<defs>" &
    "<linearGradient id='grad' x1='0' y1='0' x2='0' y2='1'>" &
    "<stop offset='0' stop-color='#0078D4'/>" &
    "<stop offset='1' stop-color='#0078D400'/>" &   -- transparent at bottom
    "</linearGradient>" &
    "</defs>"
-- Reference in fill:  fill='url(#grad)'
```

| Attribute | Description |
|-----------|-------------|
| `id` | Reference name — use in `fill='url(#id)'` |
| `x1, y1, x2, y2` | Gradient direction (0–1 normalized in `objectBoundingBox`) |
| `gradientUnits` | `objectBoundingBox` (default) or `userSpaceOnUse` |
| `stop offset` | Position along gradient (0–1 or 0%–100%) |
| `stop-color` | Color at this stop (8-digit hex for transparency) |

---

## Common Patterns

### Responsive sizing — use `viewBox` not fixed dimensions
```dax
-- Table/Matrix cells: wide and short
"<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 25'>"

-- Image visual KPI: wider aspect ratio
"<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 300 60'>"
```

### Coordinate inversion — Y=0 is at the top
```dax
VAR _Y = _ViewBoxHeight - _NormalizedValue    -- flip so higher values go up
```

### Render order — first = back, last = front
```dax
-- Correct: background → track → bar → target line → label
_SvgPrefix & _Background & _Track & _Bar & _TargetLine & _Label & _SvgSuffix
```

### Transparency via 8-digit hex
```dax
"fill='#2196F333'"   -- 33 = ~20% opacity
"fill='#FFFFFF00'"   -- 00 = fully transparent
```
