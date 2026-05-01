---
name: svg-measures
description: Reference guide for generating SVG markup via DAX measures in Power BI. Covers KPI icons, progress bars, sparklines, gauges, and traffic lights rendered as SVG strings for use with HTML Content visuals, Deneb, or Charticulator. Includes realistic DAX patterns and TMDL declarations.
version: 1.0
---

# SVG Measures for Power BI

Generate dynamic, scalable vector graphics directly from DAX measures. SVG markup returned as text strings can be displayed in HTML Content visuals, Deneb, or Charticulator, enabling custom KPI visualizations without static images.

---

## 1. How SVG Measures Work in Power BI

### The Core Pattern

A DAX measure returns a **text string containing SVG XML markup**. Power BI's HTML Content visual interprets this and renders the vector graphic in real-time, updating whenever the underlying data changes.

#### Raw SVG Approach
```dax
Simple Circle =
"<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100' viewBox='0 0 100 100'>
  <circle cx='50' cy='50' r='40' fill='#0078D4'/>
</svg>"
```

#### Data URI Approach
Encode SVG as a data URI for use in image URLs or when inline SVG is restricted:
```dax
SVG Data URI =
"data:image/svg+xml," & 
ENCODEURL("<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100'><circle cx='50' cy='50' r='40' fill='#0078D4'/></svg>")
```

### When to Use Each Approach

| Approach | When to Use | Pros | Cons |
|----------|-----------|------|------|
| **Raw SVG** | HTML Content visual, Deneb custom visual | Simple, no encoding overhead | Requires compatible visual |
| **Data URI** | Image visuals, Background images | Works in any visual type | Longer strings, ENCODEURL needed |
| **CONCATENATEX** | Complex multi-element graphics | Dynamic row-by-row generation | Performance hit on large tables |

### Performance Considerations

- Keep SVG string length under 2000 characters when possible
- Use viewBox scaling instead of multiple size attributes
- CONCATENATEX over large tables can slow refresh; filter data first
- Cache intermediate calculations using `VAR` to avoid recalculation

---

## 2. SVG Basics for DAX

### Coordinate System

SVG uses a **2D Cartesian coordinate system**:
- **Origin (0, 0)** is at the top-left
- **X increases rightward**, Y increases downward
- Units are arbitrary (pixels, relative units, etc.)
- Rotation angles are clockwise from the positive X-axis

### Common SVG Elements

#### `<svg>` — Container
```xml
<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100' viewBox='0 0 100 100'>
  <!-- content -->
</svg>
```
- **xmlns**: Required for standalone SVG
- **width/height**: Display dimensions in HTML
- **viewBox='minX minY width height'**: Internal coordinate space (enables scaling)

#### `<rect>` — Rectangle
```xml
<rect x='10' y='10' width='80' height='60' fill='#0078D4' stroke='#333' stroke-width='2'/>
```

#### `<circle>` — Circle
```xml
<circle cx='50' cy='50' r='30' fill='#2E7D32' opacity='0.8'/>
```

#### `<line>` — Line
```xml
<line x1='10' y1='10' x2='90' y2='90' stroke='#000' stroke-width='2' stroke-linecap='round'/>
```

#### `<text>` — Text
```xml
<text x='50' y='50' text-anchor='middle' dominant-baseline='middle' font-size='16' fill='#333'>
  Label
</text>
```

#### `<path>` — Path (lines, curves, shapes)
```xml
<path d='M 10 10 L 90 10 L 50 90 Z' fill='#FF6B6B' stroke='#333' stroke-width='1'/>
```
Path commands:
- **M x y**: Move to (x, y)
- **L x y**: Line to (x, y)
- **H x**: Horizontal line to x
- **V y**: Vertical line to y
- **C x1 y1 x2 y2 x y**: Cubic Bezier curve
- **Q x1 y1 x y**: Quadratic curve
- **A rx ry rotation large-arc-flag sweep-flag x y**: Arc
- **Z**: Close path

#### `<polyline>` — Connected Lines
```xml
<polyline points='10,80 20,60 30,40 40,20' fill='none' stroke='#0078D4' stroke-width='2'/>
```

#### `<g>` — Group
```xml
<g fill='#0078D4'>
  <rect x='0' y='0' width='20' height='20'/>
  <circle cx='30' cy='10' r='10'/>
</g>
```
Groups share attributes and can be transformed together.

### Style Attributes vs CSS

Common fill/stroke attributes:
```xml
fill='#0078D4'           <!-- solid color -->
fill='none'              <!-- transparent -->
stroke='#333'            <!-- outline color -->
stroke-width='2'         <!-- outline thickness -->
opacity='0.5'            <!-- transparency (0-1) -->
fill-opacity='0.7'       <!-- fill transparency only -->
stroke-linecap='round'   <!-- line end style: butt, round, square -->
stroke-linejoin='miter'  <!-- corner style: miter, round, bevel -->
```

---

## 3. KPI Icon Patterns

### Up Arrow (Positive Variance)

```dax
KPI Arrow Up =
VAR _size = 40
VAR _halfSize = _size / 2
VAR _arrowPath = "M " & _halfSize & " 5 L " & (_size - 5) & " " & (_size - 5) & " L " & _halfSize & " " & (_size - 10) & " L 5 " & (_size - 5) & " Z"
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='" & _size & "' height='" & _size & "' viewBox='0 0 " & _size & " " & _size & "'>
  <path d='" & _arrowPath & "' fill='#2E7D32' stroke='none'/>
</svg>"
```

### Down Arrow (Negative Variance)

```dax
KPI Arrow Down =
VAR _size = 40
VAR _halfSize = _size / 2
VAR _arrowPath = "M " & _halfSize & " " & (_size - 5) & " L " & (_size - 5) & " 5 L " & _halfSize & " 10 L 5 5 Z"
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='" & _size & "' height='" & _size & "' viewBox='0 0 " & _size & " " & _size & "'>
  <path d='" & _arrowPath & "' fill='#C62828' stroke='none'/>
</svg>"
```

### Conditional Arrow (Up/Down/Flat based on Threshold)

```dax
Sales Variance Icon =
VAR _variance = [Sales] - [Sales Prior Year]
VAR _color = SWITCH(
    TRUE(),
    _variance > 100, "#2E7D32",      -- green: up
    _variance < -100, "#C62828",     -- red: down
    "#FFA500"                         -- orange: flat
)
VAR _path = SWITCH(
    TRUE(),
    _variance > 100, "M 20 35 L 35 5 L 50 35 Z",           -- up arrow
    _variance < -100, "M 20 5 L 35 35 L 50 5 Z",           -- down arrow
    "M 10 20 L 50 20"                                        -- flat line
)
VAR _strokeWidth = IF(_variance < -100 OR _variance > 100, 0, 2)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='60' height='40' viewBox='0 0 60 40'>
  <path d='" & _path & "' fill='" & IF(_variance < -100 OR _variance > 100, _color, "none") & "' 
        stroke='" & _color & "' stroke-width='" & _strokeWidth & "' stroke-linecap='round'/>
</svg>"
```

---

## 4. Progress Bar / Bullet Chart

### Horizontal Progress Bar

```dax
Progress Bar =
VAR _actual = [Actual Sales]
VAR _target = [Target Sales]
VAR _percentage = DIVIDE(_actual, _target, 0)
VAR _percentage_clamped = MIN(_percentage, 1)
VAR _barWidth = _percentage_clamped * 250
VAR _color = SWITCH(
    TRUE(),
    _percentage_clamped >= 0.9, "#2E7D32",   -- green
    _percentage_clamped >= 0.7, "#FFA500",   -- orange
    "#C62828"                                 -- red
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='280' height='30' viewBox='0 0 280 30'>
  <rect x='10' y='5' width='250' height='20' fill='#E0E0E0' rx='3'/>
  <rect x='10' y='5' width='" & _barWidth & "' height='20' fill='" & _color & "' rx='3'/>
  <text x='140' y='20' text-anchor='middle' font-size='12' font-weight='bold' fill='#333'>
    " & FORMAT(_percentage, "0.0%") & "
  </text>
</svg>"
```

### Bullet Chart (Actual vs Target vs Range)

```dax
Bullet Chart =
VAR _actual = [Revenue]
VAR _target = [Revenue Target]
VAR _poor = _target * 0.5
VAR _satisfactory = _target * 0.75
VAR _good = _target * 1.0
VAR _scale = 350 / (_target * 1.25)
VAR _actualX = 50 + (_actual * _scale)
VAR _targetX = 50 + (_target * _scale)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='400' height='50' viewBox='0 0 400 50'>
  <rect x='50' y='15' width='" & (_poor * _scale) & "' height='20' fill='#FFCDD2'/>
  <rect x='50' y='15' width='" & (_satisfactory * _scale) & "' height='20' fill='#FFF9C4'/>
  <rect x='50' y='15' width='" & (_good * _scale) & "' height='20' fill='#C8E6C9'/>
  <line x1='" & _targetX & "' y1='10' x2='" & _targetX & "' y2='40' stroke='#333' stroke-width='3'/>
  <circle cx='" & _actualX & "' cy='25' r='6' fill='#0078D4'/>
</svg>"
```

---

## 5. Sparkline Pattern

### Mini Line Chart from Time-Series Data

Generate a sparkline by concatenating points from a related date table.

```dax
Revenue Sparkline =
VAR _minDate = MIN('Date'[Date])
VAR _maxDate = MAX('Date'[Date])
VAR _points = CONCATENATEX(
    FILTER(
        SUMMARIZE('Date', 'Date'[Date]),
        'Date'[Date] >= _minDate && 'Date'[Date] <= _maxDate
    ),
    VAR _dayIndex = DIVIDE(DATEDIF(_minDate, 'Date'[Date], DAY), 
                          DATEDIF(_minDate, _maxDate, DAY) * 1.0, 0) * 300
    VAR _dailyRev = CALCULATE([Revenue])
    VAR _maxRev = CALCULATE([Revenue], ALLEXCEPT('Date', 'Date'[Date]))
    VAR _yValue = 80 - DIVIDE(_dailyRev, _maxRev, 0) * 60
    RETURN
    _dayIndex & "," & _yValue & " ",
    'Date'[Date],
    ASC
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='320' height='100' viewBox='0 0 320 100'>
  <rect x='0' y='0' width='320' height='100' fill='#F5F5F5' rx='4'/>
  <polyline points='" & _points & "' fill='none' stroke='#0078D4' stroke-width='2'/>
</svg>"
```

### Smoothed Curve Sparkline

For a smoother appearance, use quadratic Bezier curves instead of polyline:

```dax
Revenue Sparkline Smooth =
VAR _points_table = SUMMARIZE('Date', 'Date'[Date], 'Date'[Week])
VAR _path = CONCATENATEX(
    _points_table,
    VAR _rev = CALCULATE([Revenue])
    VAR _x = RANKX(_points_table, 'Date'[Date], , ASC) * 15
    VAR _y = 80 - _rev / MAXX(ALLEXCEPT(_points_table, 'Date'[Week]), CALCULATE([Revenue])) * 60
    RETURN "L " & _x & " " & _y & " ",
    'Date'[Date],
    ASC
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='320' height='100' viewBox='0 0 320 100'>
  <path d='M 10 80 " & _path & "' fill='none' stroke='#2E7D32' stroke-width='2' stroke-linecap='round'/>
</svg>"
```

---

## 6. Gauge / Donut Chart

### Circular Progress Gauge

Uses SVG arc paths to draw a partial circle based on percentage.

```dax
Gauge Chart =
VAR _percentage = DIVIDE([Actual], [Target], 0)
VAR _percentage_clamped = MIN(MAX(_percentage, 0), 1)
VAR _angle = _percentage_clamped * 360
VAR _radians = _angle * PI() / 180
VAR _x = 50 + 35 * COS(_radians - PI() / 2)
VAR _y = 50 + 35 * SIN(_radians - PI() / 2)
VAR _largeArc = IF(_angle > 180, 1, 0)
VAR _color = SWITCH(
    TRUE(),
    _percentage_clamped >= 0.9, "#2E7D32",
    _percentage_clamped >= 0.7, "#FFA500",
    "#C62828"
)
VAR _centerValue = FORMAT(_percentage_clamped, "0.0%")
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120' viewBox='0 0 120 120'>
  <circle cx='60' cy='60' r='35' fill='none' stroke='#E0E0E0' stroke-width='8'/>
  <path d='M 60 25 A 35 35 0 " & _largeArc & " 1 " & _x & " " & _y & "' 
        fill='none' stroke='" & _color & "' stroke-width='8' stroke-linecap='round'/>
  <circle cx='60' cy='60' r='25' fill='white' stroke='none'/>
  <text x='60' y='60' text-anchor='middle' dominant-baseline='middle' 
        font-size='18' font-weight='bold' fill='#333'>
    " & _centerValue & "
  </text>
</svg>"
```

### Donut with Multi-Segment Breakdown

```dax
Donut Breakdown =
VAR _total = SUMX(VALUES('Category'), [Sales])
VAR _segments = CONCATENATEX(
    VALUES('Category'),
    VAR _catSales = CALCULATE([Sales])
    VAR _pct = DIVIDE(_catSales, _total, 0)
    VAR _angle = _pct * 360
    VAR _radians = _angle * PI() / 180
    VAR _x = 60 + 30 * COS(_radians - PI() / 2)
    VAR _y = 60 + 30 * SIN(_radians - PI() / 2)
    VAR _largeArc = IF(_angle > 180, 1, 0)
    VAR _color = SWITCH(
        'Category'[Category],
        "A", "#0078D4",
        "B", "#2E7D32",
        "C", "#FFA500",
        "D", "#C62828"
    )
    RETURN
    "<path d='M 60 30 A 30 30 0 " & _largeArc & " 1 " & _x & " " & _y & " L 60 60 Z' 
    fill='" & _color & "' stroke='white' stroke-width='2'/>"
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='120' height='120' viewBox='0 0 120 120'>
  " & _segments & "
</svg>"
```

---

## 7. Traffic Light / RAG Status

### Three-Circle Status Indicator

```dax
Traffic Light Status =
VAR _score = [Customer Satisfaction]
VAR _active = SWITCH(
    TRUE(),
    _score >= 80, "red",     -- red light for highest (top performer)
    _score >= 50, "amber",   -- amber for middle
    "green"                   -- green for lower
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='50' height='150' viewBox='0 0 50 150'>
  <circle cx='25' cy='25' r='15' fill='" & IF(_active = "red", "#C62828", "#E0E0E0") & "'/>
  <circle cx='25' cy='75' r='15' fill='" & IF(_active = "amber", "#FFA500", "#E0E0E0") & "'/>
  <circle cx='25' cy='125' r='15' fill='" & IF(_active = "green", "#2E7D32", "#E0E0E0") & "'/>
</svg>"
```

### Single Status Circle with Text

```dax
Status Badge =
VAR _status = [Performance Rating]
VAR _color = SWITCH(
    _status,
    "Excellent", "#2E7D32",
    "Good", "#4CAF50",
    "Fair", "#FFA500",
    "Poor", "#C62828",
    "#999999"
)
VAR _label = SWITCH(
    _status,
    "Excellent", "A",
    "Good", "B",
    "Fair", "C",
    "Poor", "D",
    "?"
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='80' height='80' viewBox='0 0 80 80'>
  <circle cx='40' cy='40' r='35' fill='" & _color & "' stroke='white' stroke-width='2'/>
  <text x='40' y='50' text-anchor='middle' font-size='36' font-weight='bold' fill='white'>
    " & _label & "
  </text>
</svg>"
```

### Multi-Threshold Bar Indicator

```dax
Threshold Bar =
VAR _value = [Metric]
VAR _threshold1 = 33
VAR _threshold2 = 67
VAR _width = DIVIDE(_value, 100, 0) * 200
VAR _color = SWITCH(
    TRUE(),
    _value <= _threshold1, "#C62828",
    _value <= _threshold2, "#FFA500",
    "#2E7D32"
)
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='220' height='30' viewBox='0 0 220 30'>
  <rect x='10' y='8' width='200' height='14' fill='#E0E0E0' rx='2'/>
  <line x1='10' y1='3' x2='10' y2='27' stroke='#999' stroke-width='1'/>
  <line x1='76.67' y1='3' x2='76.67' y2='27' stroke='#999' stroke-width='1'/>
  <line x1='143.33' y1='3' x2='143.33' y2='27' stroke='#999' stroke-width='1'/>
  <rect x='10' y='8' width='" & _width & "' height='14' fill='" & _color & "' rx='2'/>
</svg>"
```

---

## 8. Dynamic Colors

### Color by Threshold Range

```dax
Color by Performance =
VAR _performance = [Metric]
RETURN
SWITCH(
    TRUE(),
    _performance >= 90, "#2E7D32",      -- green
    _performance >= 75, "#4CAF50",      -- light green
    _performance >= 60, "#FFC107",      -- yellow
    _performance >= 45, "#FF9800",      -- orange
    _performance >= 30, "#FF6F00",      -- deep orange
    "#C62828"                            -- red
)
```

### Color by Variance Intensity

```dax
Color by Variance Intensity =
VAR _variance = [Actual] - [Target]
VAR _variance_pct = DIVIDE(ABS(_variance), [Target], 0)
VAR _isPositive = _variance >= 0
RETURN
SWITCH(
    TRUE(),
    _variance_pct >= 0.3 && _isPositive, "#1B5E20",     -- dark green
    _variance_pct >= 0.15 && _isPositive, "#2E7D32",    -- green
    _variance_pct >= 0.05 && _isPositive, "#A5D6A7",    -- light green
    _variance_pct >= 0.05 && NOT _isPositive, "#FFCDD2", -- light red
    _variance_pct >= 0.15 && NOT _isPositive, "#C62828", -- red
    "#B71C1C",                                           -- dark red
    "#9E9E9E"                                            -- gray (no variance)
)
```

### Interpolated Color Gradient

```dax
Color Gradient =
VAR _minValue = 0
VAR _maxValue = 100
VAR _currentValue = [Metric]
VAR _position = DIVIDE(_currentValue - _minValue, _maxValue - _minValue, 0)
VAR _redHex = INT(255 * (1 - _position))
VAR _greenHex = INT(255 * _position)
VAR _blueHex = 100
RETURN
"#" & 
FORMAT(_redHex, "00") & 
FORMAT(_greenHex, "00") & 
FORMAT(_blueHex, "00")
```

---

## 9. Text Overlays

### Centered Value with Unit

```dax
KPI with Text =
VAR _value = [Revenue]
VAR _formatted = FORMAT(_value, "$#,##0")
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='200' height='100' viewBox='0 0 200 100'>
  <rect x='10' y='10' width='180' height='80' fill='#F5F5F5' stroke='#0078D4' stroke-width='2' rx='4'/>
  <text x='100' y='45' text-anchor='middle' dominant-baseline='middle' 
        font-size='32' font-weight='bold' fill='#0078D4'>
    " & _formatted & "
  </text>
  <text x='100' y='70' text-anchor='middle' font-size='12' fill='#666'>
    Total Revenue
  </text>
</svg>"
```

### Stacked Text with Icon

```dax
Metric Card =
VAR _mainValue = [Sales]
VAR _comparison = [Sales PY]
VAR _variance = _mainValue - _comparison
VAR _varianceText = FORMAT(_variance, "+#,##0;-#,##0;0")
VAR _color = IF(_variance >= 0, "#2E7D32", "#C62828")
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='250' height='120' viewBox='0 0 250 120'>
  <rect x='0' y='0' width='250' height='120' fill='white' stroke='#DDD' stroke-width='1' rx='4'/>
  <text x='20' y='30' font-size='14' fill='#666'>Sales</text>
  <text x='20' y='65' font-size='28' font-weight='bold' fill='#333'>
    " & FORMAT(_mainValue, "$#,##0") & "
  </text>
  <text x='220' y='60' text-anchor='end' font-size='16' font-weight='bold' fill='" & _color & "'>
    " & _varianceText & "
  </text>
</svg>"
```

### Percentage Label in Progress Bar

```dax
Progress with Label =
VAR _actual = [Actual]
VAR _target = [Target]
VAR _pct = DIVIDE(_actual, _target, 0)
VAR _barWidth = _pct * 200
VAR _textColor = IF(_pct >= 0.5, "white", "#333")
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='240' height='40' viewBox='0 0 240 40'>
  <rect x='20' y='10' width='200' height='20' fill='#E0E0E0' rx='3'/>
  <rect x='20' y='10' width='" & _barWidth & "' height='20' fill='#0078D4' rx='3'/>
  <text x='120' y='23' text-anchor='middle' dominant-baseline='middle' 
        font-size='12' font-weight='bold' fill='" & _textColor & "'>
    " & FORMAT(_pct, "0.0%") & "
  </text>
</svg>"
```

---

## 10. Data URI Encoding

### When and Why to Use Data URIs

Use **data URIs** when:
- Embedding SVG in `<img>` tags within HTML Content visual
- Passing SVG to tools that don't accept raw markup
- Creating self-contained image URLs for export
- Reducing network requests (no external image file)

Avoid data URIs when:
- Using raw SVG directly in HTML Content visual (unnecessary encoding overhead)
- String length exceeds 2000 characters (browsers/systems may truncate)
- Measure needs to be frequently recalculated (encoding is CPU-intensive)

### Data URI Syntax

```dax
SVG as Data URI =
VAR _svgString = "<svg xmlns='http://www.w3.org/2000/svg' width='100' height='100'>
  <circle cx='50' cy='50' r='40' fill='#0078D4'/>
</svg>"
RETURN
"data:image/svg+xml," & ENCODEURL(_svgString)
```

### Data URI with Base64 Encoding

For longer SVGs, use Base64 to reduce string length (though DAX doesn't have native Base64; use alternative):

```
data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciI...
```
(Base64 encoding must be done outside DAX or via Power Query)

### Full Example: Embedding SVG in Image URL

```dax
Logo SVG =
VAR _svg = "<svg xmlns='http://www.w3.org/2000/svg' width='100' height='50' viewBox='0 0 100 50'>
  <rect x='0' y='0' width='100' height='50' fill='#0078D4'/>
  <text x='50' y='25' text-anchor='middle' dominant-baseline='middle' font-size='20' fill='white' font-weight='bold'>
    LOGO
  </text>
</svg>"
RETURN
"data:image/svg+xml," & ENCODEURL(_svg)
```

Then use in Image visual:
```
= IF(HASONEVALUE('Table'[Category]), [Logo SVG], BLANK())
```

---

## Complete Real-World Example: Executive Dashboard Card

Combining multiple patterns into a single comprehensive KPI card:

```dax
Executive KPI Card =
VAR _sales = [Total Sales]
VAR _target = [Sales Target]
VAR _variance = _sales - _target
VAR _variance_pct = DIVIDE(_variance, _target, 0)
VAR _actualPct = DIVIDE(_sales, _target, 0)
VAR _actualPct_clamped = MIN(MAX(_actualPct, 0), 1)
VAR _barWidth = _actualPct_clamped * 180
VAR _statusColor = SWITCH(
    TRUE(),
    _actualPct >= 0.95, "#2E7D32",
    _actualPct >= 0.75, "#FFA500",
    "#C62828"
)
VAR _arrowPath = IF(_variance >= 0,
    "M 210 40 L 225 25 L 240 40 Z",
    "M 210 25 L 225 40 L 240 25 Z"
)
VAR _arrowColor = IF(_variance >= 0, "#2E7D32", "#C62828")
VAR _varianceText = FORMAT(_variance, "+$#,##0;-$#,##0;$0")
RETURN
"<svg xmlns='http://www.w3.org/2000/svg' width='280' height='140' viewBox='0 0 280 140'>
  <defs>
    <filter id='shadow' x='-50%' y='-50%' width='200%' height='200%'>
      <feDropShadow dx='0' dy='2' stdDeviation='3' flood-opacity='0.15'/>
    </filter>
  </defs>
  <rect x='5' y='5' width='270' height='130' fill='white' stroke='#DDD' stroke-width='1' rx='6' filter='url(#shadow)'/>
  
  <text x='15' y='25' font-size='12' font-weight='bold' fill='#666'>SALES PERFORMANCE</text>
  
  <text x='15' y='50' font-size='24' font-weight='bold' fill='#333'>
    " & FORMAT(_sales, "$#,##0") & "
  </text>
  
  <rect x='15' y='65' width='180' height='12' fill='#E0E0E0' rx='2'/>
  <rect x='15' y='65' width='" & _barWidth & "' height='12' fill='" & _statusColor & "' rx='2'/>
  <text x='200' y='72' font-size='10' font-weight='bold' fill='#333'>
    " & FORMAT(_actualPct_clamped, "0%") & "
  </text>
  
  <text x='15' y='105' font-size='10' fill='#666'>vs Target: " & FORMAT(_target, "$#,##0") & "</text>
  <text x='15' y='120' font-size='11' font-weight='bold' fill='" & _arrowColor & "'>
    " & _varianceText & " (" & FORMAT(_variance_pct, "+0.0%;-0.0%;0.0%") & ")
  </text>
  
  <path d='" & _arrowPath & "' fill='" & _arrowColor & "'/>
</svg>"
```

---

## TMDL Measure Declaration

### Basic SVG Measure in TMDL

```tmdl
measure 'Sales Variance Icon' = 
  VAR _variance = [Sales] - [Sales Prior Year]
  VAR _color = IF(_variance >= 0, "#2E7D32", "#C62828")
  VAR _path = IF(_variance >= 0, 
    "M 20 35 L 35 5 L 50 35 Z",
    "M 20 5 L 35 35 L 50 5 Z"
  )
  RETURN
  "<svg xmlns='http://www.w3.org/2000/svg' width='60' height='40' viewBox='0 0 60 40'>" &
  "<path d='" & _path & "' fill='" & _color & "'/>" &
  "</svg>"
  formatString: ""
  displayFolder: "KPI Icons"
  description: "Displays a colored up or down arrow based on sales variance"
```

### Complex SVG Measure with Metadata

```tmdl
measure 'Revenue Sparkline' = 
  VAR _minDate = MIN('Date'[Date])
  VAR _maxDate = MAX('Date'[Date])
  VAR _points = CONCATENATEX(
    FILTER(SUMMARIZE('Date', 'Date'[Date]), 'Date'[Date] >= _minDate && 'Date'[Date] <= _maxDate),
    VAR _dayIndex = DIVIDE(DATEDIF(_minDate, 'Date'[Date], DAY), DATEDIF(_minDate, _maxDate, DAY), 0) * 300
    VAR _dailyRev = CALCULATE([Revenue])
    VAR _maxRev = CALCULATE([Revenue], ALLEXCEPT('Date', 'Date'[Date]))
    VAR _yValue = 80 - DIVIDE(_dailyRev, _maxRev, 0) * 60
    RETURN _dayIndex & "," & _yValue & " ",
    'Date'[Date], ASC
  )
  RETURN
  "<svg xmlns='http://www.w3.org/2000/svg' width='320' height='100' viewBox='0 0 320 100'>" &
  "<rect x='0' y='0' width='320' height='100' fill='#F5F5F5' rx='4'/>" &
  "<polyline points='" & _points & "' fill='none' stroke='#0078D4' stroke-width='2'/>" &
  "</svg>"
  formatString: ""
  displayFolder: "Sparklines"
  description: "Mini time-series chart showing revenue trend over selected period"
```

### Measure with Linked Annotation

```tmdl
measure 'Traffic Light Status' = 
  VAR _score = [Customer Satisfaction]
  VAR _active = SWITCH(
    TRUE(),
    _score >= 80, "red",
    _score >= 50, "amber",
    "green"
  )
  RETURN
  "<svg xmlns='http://www.w3.org/2000/svg' width='50' height='150' viewBox='0 0 50 150'>" &
  "<circle cx='25' cy='25' r='15' fill='" & IF(_active = "red", "#C62828", "#E0E0E0") & "'/>" &
  "<circle cx='25' cy='75' r='15' fill='" & IF(_active = "amber", "#FFA500", "#E0E0E0") & "'/>" &
  "<circle cx='25' cy='125' r='15' fill='" & IF(_active = "green", "#2E7D32", "#E0E0E0") & "'/>" &
  "</svg>"
  formatString: ""
  displayFolder: "Status Indicators"
  description: "Three-light traffic indicator; red=top, amber=middle, green=bottom"
```

---

## Debugging and Optimization Tips

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| SVG not rendering | Missing xmlns attribute | Add `xmlns='http://www.w3.org/2000/svg'` to root `<svg>` |
| Distorted shapes | Incorrect viewBox | Ensure viewBox width/height match aspect ratio expectations |
| Text cutoff | Dominant-baseline not set | Use `dominant-baseline='middle'` for centered text |
| Colors not showing | Quotes mismatch in DAX string | Use single quotes inside DAX, escape as needed |
| Performance lag | CONCATENATEX over large table | Filter data first; use `TOPN()` or `FILTER()` |
| String length error | SVG markup too long | Reduce detail; simplify paths; consider multiple measures |

### Optimization Checklist

- Use `VAR` to cache calculations instead of repeating expressions
- Limit decimal places in calculated coordinates: `ROUND(_x, 2)`
- Remove unnecessary whitespace in SVG markup: `"<svg>" & ... & "</svg>"` (no line breaks)
- Test with sample data first before deploying to large datasets
- Use `SELECTEDVALUE()` to reduce scope in filter contexts
- Consider pre-calculating SVG in Power Query if measure is called thousands of times per refresh

### Testing SVG Output

1. Create a simple Text visual showing `[Your Measure]`
2. Copy the measure output and paste into an online SVG viewer (e.g., SVGViewer.dev)
3. Adjust coordinates and colors until satisfied
4. Place measure in HTML Content visual for final rendering

---

## Reference: Hex Color Palette

Common brand/status colors for SVG measures:

```
Green (Success):   #2E7D32, #4CAF50, #66BB6A
Blue (Primary):    #0078D4, #1976D2, #1E88E5
Orange (Warning):  #FFA500, #FF9800, #FFB74D
Red (Alert):       #C62828, #F44336, #EF5350
Gray (Neutral):    #9E9E9E, #E0E0E0, #F5F5F5
```

---

## Links and Resources

- [MDN: SVG Guide](https://developer.mozilla.org/en-US/docs/Web/SVG)
- [SVG Path Commands](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths)
- [Power BI Custom Visuals: Deneb](https://deneb-viz.github.io/)
- [DAX String Functions](https://learn.microsoft.com/en-us/dax/concatenatex-function-dax)

---

**Version**: 1.0  
**Last Updated**: 2026-04-30  
**Author**: Power BI SVG Measures Reference  
**License**: Open Source
