# PowerBI-Claude-Design

A library of **Claude.ai skill packages** for authoring Power BI semantic models and reports — TMDL syntax, DAX measures, calculation groups, DAX UDFs, and inline SVG visual measures — plus a Contoso sample dataset and a working PBIP test project for trying everything out.

Every skill outputs **plain text TMDL/DAX** ready to paste into Power BI Desktop, [Tabular Editor](https://tabulareditor.com/), or any TMDL-aware tool. No build step, no MCP required.

---

## What's in here

```
PowerBI-Claude-Design/
├── tmdl-standards/         ← TMDL syntax foundation (start here)
├── measures/               ← DAX measure patterns & naming
├── calc-groups/            ← Calculation groups in TMDL
├── dax-udf/                ← DAX User-Defined Functions
├── svg-measures/           ← SVG-string DAX measures for inline visuals
│   ├── examples/           ← 14 ready-to-adapt .dax files
│   └── references/         ← SVG element reference + visual-specific patterns
└── datamodel/              ← Contoso sample dataset
    ├── ContosoDatacsv-10k.7z
    ├── ContosoDatacsv-10k/ ← 8 unpacked CSVs (~26 MB)
    └── contosoERD.png      ← Entity-relationship diagram
```

---

## Skill packages

Each folder is a self-contained Claude skill — drop the `SKILL.md` (and any `references/` / `examples/`) into a Claude.ai Project as knowledge files, then prompt naturally.

### `tmdl-standards/` — Foundation
Core TMDL ground rules that every other skill builds on:
- TAB-only indentation (mixed tabs/spaces = parse error)
- `createOrReplace` deployment wrapper
- Object declaration vs property syntax (`name = expr` vs `prop: value`)
- Quoting names with spaces, multi-line expression indentation
- What **not** to emit (`expression:` keyword on measures, `lineageTag`, `formatStringDefinition`)

**Always start here.** Everything below assumes these rules are in effect.

### `measures/` — DAX Measures
Production-quality DAX measure patterns:
- Naming prefixes (`#` count, `$` currency, `%` ratio, `Δ` variance, `Δ%` percent variance)
- Display-folder taxonomy (Key Metrics, Time Intelligence, Budget vs Actual, …)
- Patterns: simple sum, SUMX row-level, conditional CALCULATE+FILTER, distinct count, DIVIDE rates
- Time intelligence: YTD / QTD / MTD / PY / PY YTD / Rolling N months
- KPI variance, status flags, semi-additive (LASTNONBLANK / LASTDATE)
- Error handling with DIVIDE / IFERROR / VAR + ISBLANK
- `formatString` quick reference

### `calc-groups/` — Calculation Groups
Replace banks of repeated time-intelligence or scenario measures with a single calc group:
- TMDL nesting rules (`calculationItem` at 3 TABs, body at 5 TABs, properties at 4 TABs)
- Time Intelligence group (None / Current / YTD / QTD / MTD / PY / PY YTD / vs PY / vs PY % / Rolling 3M / Rolling 12M)
- Scenario group (Actual / Budget / Forecast / vs Budget / vs Budget %)
- Currency conversion group (USD / EUR / GBP / JPY)
- Precedence ordering (higher number = outermost wrapper)
- `ISSELECTEDMEASURE` for excluding specific measures

### `dax-udf/` — DAX User-Defined Functions
Reusable typed functions stored in the semantic model:
- Type system: `Scalar Int64/Decimal/Double/String/DateTime/Boolean/Numeric/Variant`, `Table`, `AnyRef`
- Parameter modes:
  - **`Val`** (default) — argument evaluated at call site, value substituted
  - **`Expr`** — raw expression substituted, evaluated in inner context (for `CALCULATE`, `FILTER`, time intelligence)
  - **`AnyRef`** — pass column/table/measure references unevaluated (for `VALUES`, `SAMEPERIODLASTYEAR`, `TREATAS`)
- Worked examples: `CircleArea`, `Mode`, `PriorYearValue`, `TodayAsDate`, `SplitString`

### `svg-measures/` — SVG Visual Measures
DAX measures that return `data:image/svg+xml;utf8,…` strings, rendered as images when `dataCategory: ImageUrl`:
- The strict 4-region VAR pattern: CONFIG → NORMALIZATION → SVG ELEMENTS → ASSEMBLY
- Axis normalization (raw values → fixed 0–100 SVG coordinate space)
- `HASONEVALUE` guard against subtotal rows
- The `<desc>` sort trick for sorting image columns by underlying value
- `CONCATENATEX` for sparkline / multi-point assembly
- Power BI SVG sandbox constraints — system fonts only, no external URLs, 32K char limit, single-quote XML attributes
- **14 reference DAX files in `examples/`**: sparkline, progress bar, dumbbell, bullet, overlapping bars, lollipop, IBCS bar, boxplot, jitter plot, waterfall, status pill, target bar, cache hit ratio
- **4 reference docs in `references/`**: SVG elements, table/matrix patterns, image visual patterns, card/slicer patterns

---

## Dependency map

```
tmdl-standards         ← syntax foundation
    └── measures       ← scalar DAX layer
            ├── calc-groups
            ├── dax-udf
            └── svg-measures   (also draws on calc-groups + dax-udf)
```

Read `tmdl-standards/SKILL.md` first. The other four assume those rules.

---

## Sample dataset — `datamodel/`

Contoso 10k-row sample, ready to load into Power BI:

| File | Rows | Description |
|---|---|---|
| `customer.csv` | ~10k | Customer dimension |
| `product.csv` | — | Product dimension |
| `store.csv` | — | Store dimension |
| `date.csv` | — | Date dimension |
| `currencyexchange.csv` | — | FX rates |
| `sales.csv` | — | Sales fact (header-level) |
| `orders.csv` | — | Order header |
| `orderrows.csv` | — | Order line items |
| `contosoERD.png` | — | Visual ERD of the model |

Both the unpacked CSVs (`datamodel/ContosoDatacsv-10k/`) and the original `.7z` archive are included. Use the CSVs to test calc-groups, time-intelligence patterns, currency conversion, and SVG measures end-to-end.

---

## Using the skills with Claude

1. **Create a Claude.ai Project** (or open an existing one).
2. **Add the SKILL.md files as knowledge** — at minimum `tmdl-standards/SKILL.md`, plus whichever others you need.
3. For `svg-measures`, also add the relevant files from `references/` and `examples/`.
4. **Prompt naturally**:

   > *Write a Year-over-Year growth measure for Sales Amount, following the TMDL and measures skills.*
   >
   > *Create a Time Intelligence calculation group with None / YTD / PY / PY YTD / vs PY % items.*
   >
   > *Build an SVG bullet-chart measure for the Sales table, comparing Sales Amount to Sales Target.*
   >
   > *Refactor these 12 individual time-intelligence measures into a single calc group.*

Claude will return deployable TMDL blocks (with the `createOrReplace` wrapper) ready to paste into Power BI Desktop or Tabular Editor.

---

## Conventions across all skills

- **TAB indentation** — never spaces.
- **`createOrReplace` wrapper** on every deployable block.
- **Single quotes** around any name containing spaces, dots, or apostrophes — `'Order Date'`, `'Customer''s Key'`.
- **Verbose, prefixed measure names** — `$ Revenue Net of Discounts`, not `$ Rev ND`.
- **`LASTDATE('Date'[Date])`** for rolling windows, never `TODAY()` (LASTDATE respects the visual filter).
- **`DIVIDE(num, den, 0)`** instead of `/` for safe division.
- **`HASONEVALUE` guard** on every SVG measure used in tables/matrices.

---

## Versions

| Skill | Version | Last updated |
|---|---|---|
| tmdl-standards | 3.0 | 2026-05-02 |
| measures | 2.0 | 2026-05-02 |
| calc-groups | 1.5 | 2026-05-02 |
| dax-udf | 2.0 | 2026-05-02 |
| svg-measures | 2.0 | 2026-05-02 |

---

## License

No license file is committed yet. Treat the contents as **all rights reserved** until a `LICENSE` is added — open an issue if you'd like to discuss reuse.
