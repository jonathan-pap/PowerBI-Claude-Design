# SVG Patterns for Table and Matrix Visuals

Table (`tableEx`) and Matrix (`pivotTable`) visuals are the primary target for DAX SVG measures. Configure `grid.imageHeight` and `grid.imageWidth` in visual objects to control rendering size.

**Recommended sizes:**

| Chart type | imageHeight | imageWidth |
|-----------|-------------|------------|
| Simple data bar | 16 | 100 |
| Progress bar / lollipop | 20–25 | 100 |
| Bullet / overlapping bars | 25 | 100–150 |
| Sparkline | 30 | 100 |
| Box plot / jitter | 25 | 100 |
| IBCS bar | 25 | 150 |

---

## Setup Checklist

1. Measure `dataCategory = ImageUrl` (set in TMDL or model tooling)
2. Visual `grid.imageHeight` and `grid.imageWidth` set in visual JSON objects
3. HASONEVALUE guard in the measure expression
4. `<desc>` sort trick if the column needs to be sortable

---

## The Sort Trick

Embed a zero-padded value in `<desc>` so Power BI sorts the image column by the underlying number:

```dax
VAR _Sort = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
-- Include immediately after _SvgPrefix in assembly
```

---

## Axis Normalization (Required for All Bar-Based Charts)

```dax
VAR _BarMin  = 20    -- left offset (space for labels)
VAR _BarMax  = 100   -- max x position
VAR _Scope   = ALLSELECTED( 'Table'[GroupColumn] )

VAR _MaxActual = CALCULATE( MAXX( _Scope, [Actual] ), REMOVEFILTERS( 'Table'[GroupColumn] ) )
VAR _MaxTarget = CALCULATE( MAXX( _Scope, [Target] ), REMOVEFILTERS( 'Table'[GroupColumn] ) )

VAR _AxisMax =
    IF(
        HASONEVALUE( 'Table'[GroupColumn] ),
        MAX( _MaxActual, _MaxTarget ),
        CALCULATE( MAX( [Actual], [Target] ), REMOVEFILTERS( 'Table'[GroupColumn] ) )
    ) * 1.1

VAR _AxisRange         = _BarMax - _BarMin
VAR _ActualNormalized  = DIVIDE( _Actual, _AxisMax ) * _AxisRange
VAR _TargetNormalized  = ( DIVIDE( _Target, _AxisMax ) * _AxisRange ) + _BarMin - 1
```

- `REMOVEFILTERS` on the group column ensures `_AxisMax` is consistent across all rows
- `* 1.1` prevents bars from clipping the right edge
- Target position = normalized value + left offset (`_BarMin - 1`)

---

## Pattern: Data Bar

The simplest SVG pattern. A proportional bar that fills relative to the column maximum.

```dax
Data Bar =
VAR _Value = [Sales Amount]
VAR _Max   = CALCULATE( MAX( [Sales Amount] ), REMOVEFILTERS( 'Product'[Category] ) )
VAR _W     = DIVIDE( _Value, _Max ) * 100
VAR _Color = "#5B8DBE"

RETURN
    IF( HASONEVALUE( 'Product'[Category] ),
        "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 16'>" &
        "<rect width='" & _W & "' height='16' fill='" & _Color & "' opacity='0.7' rx='2'/>" &
        "<text x='" & (_W + 3) & "' y='12' font-size='10' fill='#333'>" & FORMAT( _Value, "#,0" ) & "</text>" &
        "</svg>",
        BLANK()
    )
```

**Variants:**
- Conditional color: `VAR _Color = IF( _Value >= _Target, "#4CAF50", "#F44336" )`
- No label: remove the `<text>` element
- Rounded pill: increase `rx` to `8`

---

## Pattern: Bullet Chart with Action Dots

Actual bar, target line, baseline, and a 4-level sentiment dot. Classic SpaceParts pattern.

```dax
SVG Bullet Chart =
-- Config
VAR _Actual      = [MTD Turnover]
VAR _Target      = [MTD Turnover 1YP]
VAR _Performance = DIVIDE( _Actual - _Target, _Target )

-- Sentiment thresholds
VAR _VeryBad  = -0.05
VAR _Bad      = -0.025
VAR _Good     =  0.025
VAR _VeryGood =  0.05

-- Colors
VAR _BackgroundColor = "#F5F5F5"
VAR _BarFillColor    = "#CFCFCF"
VAR _BaselineColor   = "#737373"
VAR _TargetColor     = "#000000"
VAR _ActionDotFill   =
    SWITCH( TRUE(),
        _Performance < _VeryBad,  "#f4ae4c",
        _Performance < _Bad,      "#ffe075",
        _Performance > _VeryGood, "#2D6390",
        _Performance > _Good,     "#74B2FF",
        "#FFFFFF00"
    )

-- Chart dimensions
VAR _BarMax = 100
VAR _BarMin = 20
VAR _Scope  = ALL( 'Customers'[Key Account Name] )

-- Axis
VAR _MaxActual = CALCULATE( MAXX( _Scope, [MTD Turnover] ),     REMOVEFILTERS( 'Customers'[Key Account Name] ) )
VAR _MaxTarget = CALCULATE( MAXX( _Scope, [MTD Turnover 1YP] ), REMOVEFILTERS( 'Customers'[Key Account Name] ) )
VAR _AxisMax   =
    IF( HASONEVALUE( 'Customers'[Key Account Name] ),
        MAX( _MaxActual, _MaxTarget ),
        CALCULATE( MAX( [MTD Turnover], [MTD Turnover 1YP] ), REMOVEFILTERS( 'Customers'[Key Account Name] ) )
    ) * 1.1
VAR _AxisRange        = _BarMax - _BarMin
VAR _ActualNormalized = DIVIDE( _Actual, _AxisMax ) * _AxisRange
VAR _TargetNormalized = ( DIVIDE( _Target, _AxisMax ) * _AxisRange ) + _BarMin - 1

-- SVG
VAR _SvgPrefix  = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg'>"
VAR _Sort       = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
VAR _ActionDot  = "<circle cx='10' cy='11' r='5' fill='" & _ActionDotFill & "'/>"
VAR _BarBg      = "<rect x='" & _BarMin & "' y='2' width='" & _BarMax & "' height='80%' fill='" & _BackgroundColor & "'/>"
VAR _ActualBar  = "<rect x='" & _BarMin & "' y='5' width='" & _ActualNormalized & "' height='50%' fill='" & _BarFillColor & "'/>"
VAR _Baseline   = "<rect x='" & _BarMin & "' y='4' width='1' height='60%' fill='" & _BaselineColor & "'/>"
VAR _TargetLine = "<rect x='" & _TargetNormalized & "' y='2' width='2' height='80%' fill='" & _TargetColor & "'/>"
VAR _SvgSuffix  = "</svg>"

RETURN
    _SvgPrefix & _Sort & _ActionDot & _BarBg & _ActualBar & _Baseline & _TargetLine & _SvgSuffix
```

**Key elements:**
- Action dot at x=10 shows 4-level sentiment at a glance
- Background rect provides the bar track
- Target = thin rect (width 2px) at normalized target position
- Baseline = thin rect at the left edge of the bar area

---

## Pattern: Overlapping Bars with Variance

Two bars (actual in front, target behind) with a variance highlight and percentage label.

```dax
SVG Overlapping Bars =
-- Config
VAR _Actual      = [Actuals MTD]
VAR _Target      = [Budget MTD]
VAR _Performance = DIVIDE( _Actual - _Target, _Target )
VAR _Font        = "Segoe UI"
VAR _FontSize    = 10
VAR _FontWeight  = 600

-- Colors
VAR _ActualColor   = "#686868"
VAR _TargetColor   = "#e1dfdd"
VAR _VarianceColor = IF( _Performance < 0, "#fab005", "#2094ff" )

-- Chart dimensions
VAR _BarMax = 100
VAR _BarMin = 30
VAR _Scope  = ALLSELECTED( 'Customers'[Key Account Name] )

-- Axis
VAR _MaxActual = CALCULATE( MAXX( _Scope, [Actuals MTD] ), REMOVEFILTERS( 'Customers'[Key Account Name] ) )
VAR _MaxTarget = CALCULATE( MAXX( _Scope, [Budget MTD] ),  REMOVEFILTERS( 'Customers'[Key Account Name] ) )
VAR _AxisMax   =
    IF( HASONEVALUE( 'Customers'[Key Account Name] ),
        MAX( _MaxActual, _MaxTarget ),
        CALCULATE( MAX( [Actuals MTD], [Budget MTD] ), REMOVEFILTERS( 'Customers'[Key Account Name] ) )
    ) * 1.1
VAR _AxisRange        = _BarMax - _BarMin
VAR _ActualNormalized = DIVIDE( _Actual, _AxisMax ) * _AxisRange
VAR _TargetNormalized = DIVIDE( _Target, _AxisMax ) * _AxisRange

-- SVG
VAR _SvgPrefix   = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg'>"
VAR _Sort        = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
VAR _Icon        = "<text x='" & _BarMin - 3 & "' y='13.5' font-family='Segoe UI' font-size='6' font-weight='700' text-anchor='end' fill='" & _VarianceColor & "'>" & FORMAT( _Performance, "▲;▼;" ) & "</text>"
VAR _Label       = "<text x='" & _BarMin - 10 & "' y='15' font-family='" & _Font & "' font-size='" & _FontSize & "' font-weight='" & _FontWeight & "' text-anchor='end' fill='" & _VarianceColor & "'>" & FORMAT( _Performance, "#,##0%;#,##0%;#,##0%" ) & "</text>"
VAR _TargetBar   = "<rect x='" & _BarMin & "' y='10' width='" & _TargetNormalized & "' height='12' fill='" & _TargetColor & "'/>"
VAR _ActualBar   = "<rect x='" & _BarMin & "' y='3'  width='" & _ActualNormalized & "' height='12' fill='" & _ActualColor & "'/>"
VAR _VarianceBar = "<rect x='" & _BarMin + MIN( _ActualNormalized, _TargetNormalized ) + 1 & "' y='" & IF( _Target > _Actual, 2.9, 9 ) & "' width='" & ABS( _ActualNormalized - _TargetNormalized ) - 1 & "' height='6' fill='" & _VarianceColor & "'/>"
VAR _SvgSuffix   = "</svg>"

RETURN
    _SvgPrefix & _Sort & _Icon & _Label & _TargetBar & _ActualBar & _VarianceBar & _SvgSuffix
```

**Key details:**
- Target bar renders first (behind), actual bar on top
- Variance bar spans the gap between the two bars
- Variance bar y shifts: `IF(_Target > _Actual, 2.9, 9)` — aligns with whichever bar is shorter

---

## Pattern: Dumbbell Chart

Two circles connected by a line — actual vs target comparison.

```dax
SVG Dumbbell Chart =
VAR _Actual = [Actuals MTD]
VAR _Target = [Sales Target MTD]
VAR _W      = 100
VAR _H      = 25
VAR _Scope  = ALLSELECTED( 'Customers'[Key Account Name] )

-- Axis
VAR _MaxActual = CALCULATE( MAXX( _Scope, [Actuals MTD] ),     REMOVEFILTERS( 'Customers'[Key Account Name] ) )
VAR _MaxTarget = CALCULATE( MAXX( _Scope, [Sales Target MTD] ), REMOVEFILTERS( 'Customers'[Key Account Name] ) )
VAR _AxisMax   = IF( HASONEVALUE( 'Customers'[Key Account Name] ), MAX( _MaxActual, _MaxTarget ),
    CALCULATE( MAX( [Actuals MTD], [Sales Target MTD] ), REMOVEFILTERS( 'Customers'[Key Account Name] ) ) ) * 1.1
VAR _ActualNorm = DIVIDE( _Actual, _AxisMax ) * _W
VAR _TargetNorm = DIVIDE( _Target, _AxisMax ) * _W

-- Colors
VAR _AxisColor = "#C7C7C7"
VAR _Fill   = IF( _Actual > _Target, "#448FD6", "#D64444" )
VAR _Stroke = IF( _Actual > _Target, "#2F6698", "#982F2F" )

VAR _SvgPrefix     = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 " & _W & " " & _H & "'>"
VAR _Sort          = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
VAR _Axis          = "<line x1='0' y1='" & _H/2 & "' x2='" & _W & "' y2='" & _H/2 & "' stroke='" & _AxisColor & "'/>"
VAR _Origin        = "<circle cx='2' cy='" & _H/2 & "' r='2' fill='" & _AxisColor & "'/>"
VAR _DumbbellLine  = "<line x1='" & _ActualNorm & "' y1='" & _H/2 & "' x2='" & _TargetNorm & "' y2='" & _H/2 & "' stroke='" & _Fill & "' stroke-width='3'/>"
VAR _TargetCircle  = "<circle cx='" & _TargetNorm & "' cy='" & _H/2 & "' r='4' fill='#F5F5F5' stroke='#C7C7C7' stroke-width='1.5'/>"
VAR _ActualCircle  = "<circle cx='" & _ActualNorm & "' cy='" & _H/2 & "' r='4' fill='" & _Fill & "' stroke='" & _Stroke & "' stroke-width='1.5'/>"
VAR _SvgSuffix     = "</svg>"

RETURN
    _SvgPrefix & _Sort & _Axis & _Origin & _DumbbellLine & _TargetCircle & _ActualCircle & _SvgSuffix
```

**Render order matters:** line first → target circle → actual circle on top.

---

## Pattern: Status Pill

Rounded badge with category color and centred text label.

```dax
Status Pill =
IF( HASONEVALUE( __GROUPBY_COLUMN ),
    VAR _Status  = SELECTEDVALUE( __GROUPBY_COLUMN )
    VAR _Color   = SWITCH( _Status,
        "On Track", "#2f9e44",
        "At Risk",  "#f08c00",
        "Late",     "#e03131",
        "#000000"
    )
    VAR _SvgPrefix  = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg'>"
    VAR _Background = "<rect x='0.5' y='0.5' width='98%' height='95%' rx='15%' fill='" & _Color & "22' stroke='" & _Color & "'/>"
    VAR _Label      = "<text x='50%' y='53%' font-family='Segoe UI' font-size='10.5' font-weight='700' fill='" & _Color & "' text-anchor='middle' dominant-baseline='middle'>" & UPPER( _Status ) & "</text>"
    VAR _SvgSuffix  = "</svg>"
    RETURN _SvgPrefix & _Background & _Label & _SvgSuffix
)
```

**Color trick:** `_Color & "22"` appends `22` (hex for ~13% opacity) to create a pastel background matching the border/text color.

---

## Pattern: Lollipop Chart

Thin stem line with a proportionally-sized endpoint dot. Dot radius scales with relative value.

```dax
Lollipop Chart =
VAR _Actual      = [Sales Amount]
VAR _Target      = [Sales Target]
VAR _Performance = DIVIDE( _Actual - _Target, _Target )
VAR _BarMax = 95
VAR _BarMin = 44
VAR _Scope  = ALLSELECTED( 'Product'[Category] )

VAR _MaxInScope   = CALCULATE( MAXX( _Scope, [Sales Amount] ), REMOVEFILTERS( 'Product'[Category] ) )
VAR _AxisMax      = IF( HASONEVALUE( 'Product'[Category] ), _MaxInScope ) * 1.1
VAR _AxisRange    = _BarMax - _BarMin
VAR _ActualNorm   = DIVIDE( _Actual, _AxisMax ) * _AxisRange
VAR _DotRadius    = MAX( DIVIDE( _Actual, _AxisMax ) * 7.5, 3.5 )   -- scales with value, min 3.5

VAR _Color = SWITCH( TRUE(), _Performance < 0, "#ffd43b", _Performance > 0, "#a5d8ff", "#CACACA" )
VAR _Label = SWITCH( TRUE(),
    _Actual <= 1E3,  FORMAT( _Actual, "#,0" ),
    _Actual <= 1E6,  FORMAT( _Actual, "#,0, K" ),
    _Actual <= 1E9,  FORMAT( _Actual, "#,0,, M" ),
    FORMAT( _Actual, "#,0,,, bn" )
)

VAR _SvgPrefix  = "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg'>"
VAR _Sort       = "<desc>" & FORMAT( _Actual, "000000000000" ) & "</desc>"
VAR _Stem       = "<rect x='" & _BarMin & "' y='10' width='" & _ActualNorm & "' height='15%' fill='" & _Color & "'/>"
VAR _Dot        = "<circle cx='" & _ActualNorm + _BarMin & "' cy='11.75' r='" & _DotRadius & "' fill='" & _Color & "'/>"
VAR _ValueLabel = "<text x='40' y='16' font-family='Segoe UI' font-size='11' font-weight='600' text-anchor='end' fill='" & _Color & "'>" & _Label & "</text>"
VAR _SvgSuffix  = "</svg>"

RETURN _SvgPrefix & _Sort & _Stem & _Dot & _ValueLabel & _SvgSuffix
```

---

## Pattern: Sparkline (Polyline)

Line sparkline showing trend over the last 12 months.

```dax
Sparkline SVG =
VAR _Values =
    ADDCOLUMNS(
        CALCULATETABLE( VALUES( 'Date'[Month] ), DATESINPERIOD( 'Date'[Date], MAX( 'Date'[Date] ), -12, MONTH ) ),
        "@Value", [Sales Amount]
    )
VAR _XMin = MIN( 'Date'[Month] )
VAR _XMax = MAX( 'Date'[Month] )
VAR _YMin = MINX( _Values, [@Value] )
VAR _YMax = MAXX( _Values, [@Value] )

VAR _Points = CONCATENATEX(
    ADDCOLUMNS( _Values,
        "@X", INT( 100 * DIVIDE( 'Date'[Month] - _XMin, _XMax - _XMin ) ),
        "@Y", INT(  30 * DIVIDE( [@Value]       - _YMin, _YMax - _YMin ) )
    ),
    [@X] & "," & ( 30 - [@Y] ),   -- invert Y
    " ",
    'Date'[Month]
)

RETURN
    IF( HASONEVALUE( 'Product'[Category] ),
        "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 30'>" &
        "<polyline fill='none' stroke='#5B8DBE' stroke-width='2' points='" & _Points & "'/>" &
        "</svg>",
        BLANK()
    )
```

---

## Pattern: Area Sparkline with Gradient

Filled area sparkline using a `<linearGradient>`.

```dax
Area Sparkline =
VAR _Defs   =
    "<defs><linearGradient id='g' x1='0' y1='0' x2='0' y2='1'>" &
    "<stop stop-color='#0078D4' offset='0'/>" &
    "<stop stop-color='#0078D400' offset='1'/>" &
    "</linearGradient></defs>"

VAR _XMin = MIN( 'Date'[Date] )
VAR _XMax = MAX( 'Date'[Date] )
VAR _Values = SUMMARIZE( 'Date', 'Date'[Date] )
VAR _YMin = MINX( _Values, CALCULATE( [Sales Amount] ) )
VAR _YMax = MAXX( _Values, CALCULATE( [Sales Amount] ) )

VAR _Points = CONCATENATEX(
    ADDCOLUMNS( _Values,
        "@X", INT( 150 * DIVIDE( 'Date'[Date] - _XMin, _XMax - _XMin ) ),
        "@Y", INT(  50 * DIVIDE( CALCULATE( [Sales Amount] ) - _YMin, _YMax - _YMin ) )
    ),
    [@X] & "," & ( 50 - [@Y] ),
    " ",
    'Date'[Date]
)

RETURN
    IF( HASONEVALUE( 'Product'[Category] ),
        "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 150 50'>" &
        _Defs &
        "<polyline fill='url(#g)' fill-opacity='0.3' stroke='#0078D4' stroke-width='2' points='0 50 " & _Points & " 150 50'/>" &
        "</svg>",
        BLANK()
    )
```

---

## Pattern: Bar Sparkline

Mini column chart showing relative bar heights across periods.

```dax
Bar Sparkline =
VAR _Values = ADDCOLUMNS(
    CALCULATETABLE( VALUES( 'Date'[Month] ), DATESINPERIOD( 'Date'[Date], MAX( 'Date'[Date] ), -12, MONTH ) ),
    "@Value", [Sales Amount]
)
VAR _Count = COUNTROWS( _Values )
VAR _Max   = MAXX( _Values, [@Value] )
VAR _Min   = MINX( _Values, [@Value] )
VAR _Range = _Max - _Min
VAR _W     = 100
VAR _H     = 30
VAR _BarW  = _W / _Count
VAR _Scale = DIVIDE( _H, _Range )

VAR _Bars = CONCATENATEX(
    ADDCOLUMNS( _Values, "@Idx", RANKX( _Values, 'Date'[Month], , ASC ) ),
    VAR _X    = ( [@Idx] - 1 ) * _BarW
    VAR _BarH = ( [@Value] - _Min ) * _Scale
    VAR _Y    = _H - _BarH
    RETURN "<rect x='" & _X & "' y='" & _Y & "' width='" & ( _BarW * 0.8 ) & "' height='" & _BarH & "' fill='#2196F3' opacity='0.7'/>",
    "", [@Idx], ASC
)

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 " & _W & " " & _H & "'>" & _Bars & "</svg>"
```

---

## UDF Calling Pattern

For reusable SVG charts, DAX UDFs encapsulate the measure so you only pass parameters:

```dax
-- Example call to a bullet chart UDF
SVG.Chart.BulletChart.ActionDot(
    [MTD Turnover],           -- Actual
    [MTD Turnover 1YP],       -- Target
    'Customers'[Key Account Name],
    -0.050,  -0.025,           -- Very bad / bad thresholds
     0.025,   0.050,           -- Good / very good thresholds
    "#f4ae4c", "#ffe075",      -- Very bad / bad colors
    "#74B2FF", "#2D6390"       -- Good / very good colors
)
```

See `dax-udf/SKILL.md` for how to declare UDF functions in TMDL.
