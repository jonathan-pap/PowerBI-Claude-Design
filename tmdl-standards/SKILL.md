---
name: tmdl-standards
description: >
  TMDL syntax ground rules for Power BI semantic models. Use this whenever
  writing or reviewing TMDL for tables, columns, measures, DAX UDFs, or
  calculation groups. Always consult before generating any TMDL — these rules
  govern indentation, object declaration, property syntax, and measure formatting.
  For UDF type system and examples see dax-udf/SKILL.md.
  For calculation group patterns see calc-groups/SKILL.md.
version: 3.0
---

# TMDL Syntax Standards

TMDL (Tabular Model Definition Language) is Power BI's indentation-based model
definition format. Every object is declared once, properties use colon syntax,
expressions use equals syntax, and indentation is always **TABs — never spaces**.

---

## Core Syntax Rules

1. **TAB indentation only** — Never spaces. Mixed tabs/spaces cause parse errors.
2. **Object declaration**: `objectType objectName` or `objectType 'Name With Spaces'`
3. **Properties**: `propertyName: value` (colon-delimited)
4. **Expressions**: `objectType Name = expression` or `property =\n\t<expression>` (equals-delimited)
5. **Single quotes** around names containing spaces, dots, colons, equals, or apostrophes
   → `'Order Date'`, `'Sales YTD'`, `'Customer''s Key'` (double the apostrophe to escape)
6. **Boolean shortcut**: `isHidden` alone means `isHidden: true` — no colon needed
7. **Descriptions**: Triple-slash `///` on the line immediately above the object declaration
8. **Multi-line expressions**: Indent one extra level beyond the property line
9. **Case**: Keywords use camelCase (`dataType`, `formatString`, `isHidden`). Parsing is case-insensitive but camelCase is convention.
10. **Comments**: Use `//` for inline comments inside expression blocks

---

## Table Structure

A table file contains the table declaration, columns, and measures. Measures may
also live in a separate dedicated `_Measures` table.

```tmdl
/// Sales transaction facts
table Sales

	column SalesKey
		dataType: int64
		isHidden
		sourceColumn: SalesKey
		summarizeBy: none

	column 'Order Date'
		dataType: dateTime
		formatString: "yyyy-mm-dd"
		sourceColumn: OrderDate
		summarizeBy: none

	column SalesAmount
		dataType: decimal
		formatString: "$#,0.00"
		sourceColumn: SalesAmount
		summarizeBy: sum

	measure 'Total Sales' = SUM(Sales[SalesAmount])
		formatString: "$#,0.00"
```

### Column Data Types

| TMDL type | Description |
|-----------|-------------|
| `string` | Text |
| `int64` | 64-bit integer |
| `decimal` | High-precision decimal (financial data) |
| `double` | Floating-point |
| `boolean` | True/False |
| `dateTime` | Date and time |
| `date` | Date only |

### Key Column Attributes

```tmdl
column MonthName
	dataType: string
	sourceColumn: MonthName
	summarizeBy: none
	sortByColumn: MonthNumber     ← sort a text column by a numeric column

column CustomerKey
	dataType: int64
	isHidden                      ← hide from report field list
	sourceColumn: CustomerKey
	summarizeBy: none             ← prevent implicit aggregation
```

**summarizeBy values**: `none`, `sum`, `average`, `count`, `min`, `max`

---

## Measure Syntax

All measures are deployed with a `createOrReplace` block targeting the `_measures`
table. The nesting is:

```
createOrReplace               ← root, no indent
	ref table _measures       ← 1 TAB
		measure 'Name' = …    ← 2 TABs  (inline DAX after =)
		— or for multi-line —
		measure 'Name' =      ← 2 TABs
				DAX body      ← 4 TABs  (jumps TWO levels — no intermediate row)
			property: …       ← 3 TABs  (properties come AFTER the body)
```

> There is no `expression:` keyword for measures in a `createOrReplace` block.
> The expression follows `=` directly on the measure name line (inline) or on the
> next line at 4 TABs (multi-line). Properties sit at 3 TABs **after** the body.

### Canonical Full Example

```tmdl
createOrReplace

	ref table _measures

		/// Total revenue from all sales transactions
		measure '$ Revenue' = SUM(Sales[SalesAmount])
			formatString: "$#,0.00"
			displayFolder: "Revenue"

		/// Year-over-year growth as a percentage
		measure '% Revenue vs PY' =
				VAR _current = [$ Revenue]
				VAR _py =
					CALCULATE(
						[$ Revenue],
						SAMEPERIODLASTYEAR('Date'[Date])
					)
				RETURN DIVIDE(_current - _py, _py)
			formatString: "0.0%"
			displayFolder: "Time Intelligence"
```

When showing individual measure bodies in other examples the `createOrReplace` /
`ref table _measures` wrapper is omitted for brevity — always include it in
generated output.

---

## DAX UDFs in TMDL

UDFs use a `createOrReplace` block with the function declaration directly at
1 TAB — no `ref table` wrapper. The parameter list and body sit 2 levels deeper
(3 TABs):

```tmdl
createOrReplace

	/// Evaluate any measure in the prior year
	function 'PriorYearValue' =
			( expression : Scalar Variant Expr, dateColumn : AnyRef ) =>
			CALCULATE(expression, SAMEPERIODLASTYEAR(dateColumn))

	/// Simple scalar — no context shift needed, Val mode is fine
	function 'CircleArea' =
			( radius : Scalar Numeric ) =>
			PI() * radius * radius
```

> For the full UDF type system (Scalar subtypes, Val/Expr modes, AnyRef),
> see **dax-udf/SKILL.md**.

---

## Calculation Group Tables

Calculation groups use `createOrReplace` with a `table` declaration at 1 TAB.
`calculationItem` entries are **nested inside `calculationGroup` at 3 TABs**.
`column` entries are siblings of `calculationGroup` at **2 TABs**.

Multi-line `calculationItem` uses `=` on the declaration line — **no `expression:`
property**. DAX body sits 2 levels deeper (5 TABs). Properties (`ordinal:`) sit
at 4 TABs. `sourceColumn: Name` — no brackets. Always include an `Ordinal` column.

```tmdl
createOrReplace

	table 'Time Intelligence'

		calculationGroup
			precedence: 10

			calculationItem 'None' = SELECTEDMEASURE()

			calculationItem 'YTD' =
					CALCULATE(
						SELECTEDMEASURE(),
						DATESYTD('Date'[Date])
					)
				ordinal: 2

		column Period
			dataType: string
			isDefaultLabel
			sourceColumn: Name
			sortByColumn: Ordinal
			summarizeBy: none

		column Ordinal
			dataType: int64
			formatString: 0
			sourceColumn: Ordinal
			summarizeBy: sum
```

> For the complete calculation group patterns (Time Intelligence, Scenario,
> Currency, precedence rules, None item), see **calc-groups/SKILL.md**.

---

## Naming Conventions

| Object | Convention | Examples |
|--------|-----------|---------|
| Fact tables | PascalCase, Fact prefix | `FactSales`, `FactInventory` |
| Dimension tables | PascalCase, Dim prefix | `DimCustomer`, `DimProduct` |
| Calculated tables | PascalCase | `_Measures`, `CalcOffsets` |
| Columns | PascalCase | `SalesKey`, `OrderDate`, `IsArchived` |
| Measures | `'Title Case In Quotes'` | `'Total Sales'`, `'Sales vs PY %'` |
| Display folders | Title Case | `"Time Intelligence"`, `"KPIs"` |

Measures with spaces in their names **must** be wrapped in single quotes in both
the TMDL declaration and in DAX references: `[Total Sales]`.

---

## formatString Quick Reference

| Pattern | Result |
|---------|--------|
| `"$#,0.00"` | $1,234.56 |
| `"€#,0.00"` | €1,234.56 |
| `"#,0"` | 1,235 |
| `"0.0%"` | 12.3% |
| `"0.00%"` | 12.35% |
| `"yyyy-mm-dd"` | 2026-01-31 |
| `"mmm yyyy"` | Jan 2026 |
| `"Short Date"` | 1/31/2026 |

---

## Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| TMDL parse error | Mixed tabs and spaces | Use only TABs; check all indentation |
| Measure shows error | Invalid table/column name | Wrap names with spaces in single quotes: `Sales[Order Date]` |
| UDF won't evaluate in context | Wrong parameter mode | Use `Expr` when the argument must be re-evaluated inside CALCULATE |
| Calc group format wrong | Hardcoded format string | Use `SELECTEDMEASUREFORMATSTRING()` unless format genuinely changes |

---

## Related References

- **measures/SKILL.md** — DAX measure patterns, naming conventions, format strings, display folders
- **calc-groups/SKILL.md** — Calculation group TMDL structure, calculationItem syntax, precedence
- **dax-udf/SKILL.md** — DAX User-Defined Function syntax, parameter types (Val/Expr/AnyRef)
- **svg-measures/SKILL.md** — SVG DAX measures, TMDL declaration patterns for image-type measures

---

**Version:** 3.0
**Last Updated:** 2026-05-02
