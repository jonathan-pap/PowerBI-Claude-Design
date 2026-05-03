# SVG Patterns for Card and Slicer Visuals

Card (`cardVisual`) and Slicer (`advancedSlicerVisual`) visuals support SVG measures through specific binding patterns.

> **Classic card (`card`) does NOT support SVG.** Only the new card visual (`cardVisual`) works. If SVG renders as raw text or blank, verify the visual type is `cardVisual`.

---

## Card Visual (`cardVisual`)

### Binding

Card visuals render SVG via `callout.imageFX`. Bind the SVG measure to the card's callout value field, then enable `imageFX` in the visual JSON objects:

```json
{
  "objects": {
    "callout": [{
      "properties": {
        "imageFX":    { "expr": { "Literal": { "Value": "true"  } } },
        "imageHeight": { "expr": { "Literal": { "Value": "40D"  } } },
        "imageWidth":  { "expr": { "Literal": { "Value": "100D" } } }
      }
    }]
  }
}
```

### Pattern: Arrow Indicator

Compact up/down directional indicator.

```dax
Arrow Indicator =
VAR _Growth = [Growth %]
VAR _Up     = _Growth >= 0
VAR _Path   = IF( _Up,
    "M 10,15 L 5,10 L 15,10 Z",   -- up triangle
    "M 10,5 L 5,10 L 15,10 Z"     -- down triangle
)
VAR _Color  = IF( _Up, "#4CAF50", "#F44336" )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20'>" &
    "<path d='" & _Path & "' fill='" & _Color & "'/></svg>"
```

### Pattern: Mini Gauge (Semi-Circle)

Semi-circular needle gauge for progress or performance.

```dax
Mini Gauge =
VAR _Pct   = DIVIDE( [Value], [Target], 0 )
VAR _Pct_c = MIN( MAX( _Pct, 0 ), 1 )           -- clamp 0–1
VAR _Angle = _Pct_c * 180 - 90                   -- -90° (left) to +90° (right)
VAR _R     = 40
VAR _CX    = 50
VAR _CY    = 50
VAR _Rad   = _Angle * PI() / 180
VAR _NX    = _CX + _R * 0.8 * COS( _Rad )
VAR _NY    = _CY + _R * 0.8 * SIN( _Rad )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 60'>" &
    -- Track
    "<path d='M 10 50 A 40 40 0 0 1 90 50' fill='none' stroke='#E0E0E0' stroke-width='8'/>" &
    -- Needle
    "<line x1='" & _CX & "' y1='" & _CY & "' x2='" & _NX & "' y2='" & _NY & "' stroke='#333' stroke-width='2'/>" &
    -- Pivot dot
    "<circle cx='" & _CX & "' cy='" & _CY & "' r='3' fill='#333'/>" &
    "</svg>"
```

**How the arc works:**
- Track: `M 10 50 A 40 40 0 0 1 90 50` — semi-circle from (10,50) to (90,50)
- Arc flags: `large-arc=0`, `sweep=1` (clockwise)
- Needle: a line from center to a point on the arc, calculated with COS/SIN

### Pattern: Mini Donut

Percentage completion as a donut ring.

```dax
Mini Donut =
VAR _Pct      = [Percentage]   -- 0 to 1
VAR _Angle    = _Pct * 360
VAR _LargeArc = IF( _Angle > 180, 1, 0 )
VAR _R        = 40
VAR _CX       = 50
VAR _CY       = 50
VAR _Rad      = ( _Angle - 90 ) * PI() / 180    -- start from top (12 o'clock)
VAR _EndX     = _CX + _R * COS( _Rad )
VAR _EndY     = _CY + _R * SIN( _Rad )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'>" &
    -- Background ring
    "<circle cx='" & _CX & "' cy='" & _CY & "' r='" & _R & "' fill='none' stroke='#E0E0E0' stroke-width='8'/>" &
    -- Progress arc (starts at top: 50, 10)
    "<path d='M " & _CX & " " & (_CY - _R) & " A " & _R & " " & _R & " 0 " & _LargeArc & " 1 " & _EndX & " " & _EndY & "' fill='none' stroke='#2196F3' stroke-width='8'/>" &
    "</svg>"
```

**Arc math:**
- Start at top: `M _CX (_CY - _R)` — directly above center
- `sweep=1` = clockwise
- `large-arc = 1` when angle > 180°, so the arc takes the long path

### Pattern: Progress Bar (Card Size)

Compact horizontal progress bar with percentage text.

```dax
Progress Bar =
VAR _Pct   = [Completion %]   -- 0 to 1
VAR _W     = 100
VAR _H     = 20
VAR _FillW = MIN( _Pct, 1 ) * _W
VAR _Label = FORMAT( _Pct, "0%" )
VAR _Color = SWITCH( TRUE(), _Pct < 0.5, "#F44336", _Pct < 0.8, "#FF9800", "#4CAF50" )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 " & _W & " " & _H & "'>" &
    "<rect width='" & _W    & "' height='" & _H & "' fill='#E0E0E0' rx='" & (_H/2) & "'/>" &
    "<rect width='" & _FillW & "' height='" & _H & "' fill='" & _Color & "' rx='" & (_H/2) & "'/>" &
    "<text x='" & (_W/2) & "' y='" & (_H/2+4) & "' font-size='11' text-anchor='middle' fill='white' font-weight='bold'>" & _Label & "</text>" &
    "</svg>"
```

---

## Slicer Visual (`advancedSlicerVisual`)

Slicers can display SVG in header images and custom slicer items. Most commonly used for branded slicer headers.

### Binding

Set SVG in slicer header via `header.image` in the visual JSON:

```json
{
  "objects": {
    "header": [{
      "properties": {
        "image": {
          "expr": {
            "Measure": {
              "Expression": { "SourceRef": { "Entity": "Table" } },
              "Property": "Header SVG"
            }
          }
        }
      }
    }]
  }
}
```

---

## Card SVG Sizing Guide

Card visuals have limited space. Keep SVGs compact:

| Indicator type | Recommended viewBox |
|---------------|---------------------|
| Arrow indicator | `0 0 20 20` |
| Mini gauge | `0 0 100 60` |
| Mini donut | `0 0 100 100` |
| Progress bar | `0 0 100 20` |

---

## Conditional Color Schemes

### 3-level traffic light (KPI)

```dax
VAR _Color = SWITCH( TRUE(),
    _Performance >= 1.0,  "#4CAF50",   -- green: exceeds target
    _Performance >= 0.8,  "#FF9800",   -- amber: near target
    "#F44336"                           -- red: below target
)
```

### 4-level sentiment (SpaceParts)

```dax
VAR _Color = SWITCH( TRUE(),
    _Performance < -0.05,  "#f4ae4c",   -- dark yellow: very bad
    _Performance < -0.025, "#ffe075",   -- light yellow: bad
    _Performance >  0.05,  "#2D6390",   -- dark blue: very good
    _Performance >  0.025, "#74B2FF",   -- light blue: good
    "#CCCCCC"                            -- grey: neutral
)
```
