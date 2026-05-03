# SVG Patterns for Image Visuals

Image visuals (`image` visual type) render an SVG measure as a standalone graphic on the report canvas. Unlike table/matrix SVGs (which are inline micro-charts per row), image visuals occupy their own visual container and can be any size.

---

## Critical: `sourceType` Must Be `'imageData'`

For `data:image/svg+xml;utf8,...` strings, the image visual **must** use `sourceType = 'imageData'`. Using `'imageUrl'` (which is for HTTP URLs) renders black or throws `VisualDataProxyExecutionUnknownError`.

---

## JSON Binding

The image visual's `visual.json` file binds the SVG measure via `objects.image`:

```json
{
  "objects": {
    "image": [{
      "properties": {
        "sourceType": { "expr": { "Literal": { "Value": "'imageData'" } } },
        "transparency": { "expr": { "Literal": { "Value": "0D" } } },
        "effects": { "expr": { "Literal": { "Value": "false" } } },
        "sourceField": {
          "expr": {
            "Measure": {
              "Expression": {
                "SourceRef": { "Entity": "Sales" }
              },
              "Property": "KPI Header SVG"
            }
          }
        }
      }
    }]
  }
}
```

**Notes:**
- Image visuals need **no `query` block** — only `objects.image` with `sourceType`, `sourceField`, and optionally `transparency` / `effects`
- Adding a `query` block can cause duplicate measure evaluations
- `transparency: 0D` and `effects: false` prevent unwanted default styling

For extension measures (report-layer), the `SourceRef` uses `"Schema": "extension"`:
```json
"SourceRef": { "Schema": "extension", "Entity": "Sales" }
```

---

## viewBox Sizing for Image Visuals

Image visuals are flexible in size. Match the viewBox to the visual's aspect ratio:

| Use Case | Recommended viewBox | Typical Visual Size |
|----------|-------------------|---------------------|
| KPI card | `0 0 300 60` | ~300×60 px |
| Sparkline | `0 0 300 50` | ~300×50 px |
| Dashboard tile | `0 0 200 80` | ~200×80 px |
| Full-width banner | `0 0 600 40` | ~600×40 px |
| Needle gauge | `0 0 320 60` | ~320×60 px |

---

## Pattern: KPI Header Card

A standalone SVG showing metric value, label, and trend indicator. Designed for image visuals at the top of a dashboard page.

```dax
KPI Header SVG =
VAR _Value       = [Total Revenue]
VAR _PY          = [Total Revenue PY]
VAR _Change      = DIVIDE( _Value - _PY, _PY )
VAR _ChangeLabel = FORMAT( _Change, "+#,##0.0%;-#,##0.0%" )
VAR _ValueLabel  = FORMAT( _Value, "$#,##0,, M" )
VAR _ChangeColor = IF( _Change >= 0, "#2D6A2E", "#982F2F" )

VAR _Arrow =
    IF( _Change >= 0,
        "<polygon points='170,28 175,20 180,28' fill='" & _ChangeColor & "'/>",
        "<polygon points='170,20 175,28 180,20' fill='" & _ChangeColor & "'/>"
    )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 300 60' font-family='Segoe UI, Arial, sans-serif'>" &
    "<text x='10' y='20' font-size='11' fill='#666' font-weight='600'>TOTAL REVENUE</text>" &
    "<text x='10' y='48' font-size='28' fill='#333' font-weight='700'>" & _ValueLabel & "</text>" &
    _Arrow &
    "<text x='185' y='28' font-size='12' fill='" & _ChangeColor & "' font-weight='600'>" & _ChangeLabel & " vs PY</text>" &
    "</svg>"
```

**Design notes:**
- Wide viewBox (`300 × 60`) since image visuals are typically wider than table cells
- All text labels inside the SVG — no separate visual title needed
- Font sizes can be larger (28px value vs 10–12px in tables)

---

## Pattern: Sparkline with Endpoint Dot

A clean sparkline with a highlighted endpoint. Good for dashboard headers alongside KPI values.

```dax
Sparkline with Endpoint =
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
        "@X", INT( 280 * DIVIDE( 'Date'[Month] - _XMin, _XMax - _XMin ) ) + 10,
        "@Y", INT(  50 * DIVIDE( [@Value]       - _YMin, _YMax - _YMin ) )
    ),
    [@X] & "," & ( 55 - [@Y] ),
    " ",
    'Date'[Month]
)

-- Calculate endpoint dot position
VAR _LastX   = 290
VAR _LastVal = MAXX( FILTER( _Values, 'Date'[Month] = _XMax ), [@Value] )
VAR _LastY   = 55 - INT( 50 * DIVIDE( _LastVal - _YMin, _YMax - _YMin ) )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 300 60'>" &
    "<polyline fill='none' stroke='#448FD6' stroke-width='2' points='" & _Points & "'/>" &
    "<circle cx='" & _LastX & "' cy='" & _LastY & "' r='4' fill='#448FD6'/>" &
    "</svg>"
```

---

## Pattern: Multi-Metric Dashboard Tile

Combines a value, progress bar, and percentage label in a single image visual tile.

```dax
Dashboard Tile SVG =
VAR _Revenue  = [Total Revenue]
VAR _Target   = [Revenue Target]
VAR _Pct      = DIVIDE( _Revenue, _Target )
VAR _RevLabel = FORMAT( _Revenue, "$#,0,, M" )
VAR _PctLabel = FORMAT( _Pct, "0%" )
VAR _BarW     = MIN( _Pct, 1 ) * 180

VAR _BarColor = SWITCH( TRUE(), _Pct < 0.5, "#F44336", _Pct < 0.8, "#FF9800", "#4CAF50" )

RETURN
    "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 200 80' font-family='Segoe UI, Arial, sans-serif'>" &

    -- Header label
    "<text x='10' y='16' font-size='10' fill='#999' font-weight='600'>REVENUE vs TARGET</text>" &

    -- Main value
    "<text x='10' y='42' font-size='22' fill='#333' font-weight='700'>" & _RevLabel & "</text>" &

    -- Progress track + fill
    "<rect x='10' y='52' width='180' height='8' fill='#E8E8E8' rx='4'/>" &
    "<rect x='10' y='52' width='" & _BarW & "' height='8' fill='" & _BarColor & "' rx='4'/>" &

    -- Percentage label below bar
    "<text x='10' y='72' font-size='10' fill='" & _BarColor & "' font-weight='600'>" & _PctLabel & " of target</text>" &

    "</svg>"
```

---

## Design Considerations

### Fonts
Use system fonts only — `Segoe UI, Arial, Calibri, sans-serif`. External fonts do not load in Power BI's SVG sandbox.

### Colors
Always use hex with `#` — e.g., `fill='#2196F3'`. Never `%23`. Named colors are unreliable.

### No Query Block
Image visuals bound to SVG measures do not need a `query` block. The measure is evaluated directly via `sourceField`. Adding one can cause duplicate evaluations.

### Disable Effects
Set `transparency: 0D` and `effects: false` on the image visual to prevent Power BI from applying default shadow or glow effects.
