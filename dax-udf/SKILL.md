---
name: dax-udf
description: >
  Reference for DAX User-Defined Functions (UDFs) in Power BI semantic models.
  Use this when building reusable function definitions with type safety and
  parameter modes (Val/Expr/AnyRef). Always consult before writing repeated
  DAX logic that could be abstracted into a single function — UDFs eliminate
  boilerplate and make complex patterns like time intelligence, statistical
  functions, and table filters reusable across the entire model.
version: 2.0
---

# DAX User-Defined Functions (UDFs)

DAX UDFs are native function definitions stored in the semantic model. Define
once, reuse across any measure. They provide type safety, parameter modes, and
support scalar, table, or reference returns.

---

## TMDL Wrapper

All UDFs are deployed with a `createOrReplace` block. The function declaration
sits at 1 TAB; the parameter list and body sit at 3 TABs:

```tmdl
createOrReplace

	/// Description of what the function does
	function 'FunctionName' =
			( param1 : Type Subtype Mode, param2 : Type Subtype Mode ) =>
			<function body>
```

Parameterless functions:

```tmdl
createOrReplace

	function 'TodayAsDate' =
			() =>
			TREATAS({ TODAY() }, 'Date'[Date])
```

---

## DAX Signature Syntax

```
( param1 [: Type [Subtype] [Val|Expr]], param2 [...] ) =>
    <Function body>
```

Parameters are optional. Parameterless functions use empty parentheses `()`.

---

## Type System

### Scalar
Single value. Specify a subtype for type safety:

| Subtype | Description |
|---------|-------------|
| `Int64` | 64-bit integer |
| `Decimal` | Fixed-point decimal |
| `Double` | Floating-point |
| `String` | Text |
| `DateTime` | Date/time |
| `Boolean` | TRUE/FALSE |
| `Numeric` | Int64, Decimal, or Double (flexible) |
| `Variant` | Any scalar type — avoid unless necessary |

`BLANK()` is valid for any scalar subtype.

### Table
A DAX table expression — rows and columns returned.

### AnyRef
Direct reference to a model object **without pre-evaluation**. Use when
passing columns, tables, or measures to DAX functions that expect references.

Allowed forms: `'Table'[Column]`, `'Table'`, `[Measure]`, named calendars.

---

## Parameter Modes

### Val (default)
Argument is **evaluated at the call site** before entering the function.
The computed value is substituted wherever the parameter appears.

```
(amount : Scalar Decimal Val) =>
    amount * 1.1
```

Use for: arithmetic, string operations, simple transformations.

### Expr
The **raw expression is substituted into the function body** unevaluated.
It is evaluated in the function's inner context — critical when the expression
must be re-evaluated inside CALCULATE, FILTER, or an iteration function.

```
(expression : Scalar Variant Expr) =>
    CALCULATE(expression, SAMEPERIODLASTYEAR('Date'[Date]))
```

Use for: measures passed to CALCULATE, context-shifting patterns, time intelligence.

> **Rule of thumb:** If you need the expression to "see" the inner filter context
> the function creates, use `Expr`. If you just need the value, use `Val`.

---

## Examples

The examples below show the complete TMDL including the `createOrReplace` wrapper.

### 1. CircleArea — Simple Scalar

```tmdl
createOrReplace

	function 'CircleArea' =
			( radius : Scalar Numeric ) =>
			PI() * radius * radius
```

Usage: `CircleArea(5)` → ~78.54

### 2. Mode — AnyRef for Column Reference

Returns the most frequently occurring value in a column:

```tmdl
createOrReplace

	function 'Mode' =
			( tab : AnyRef, col : AnyRef ) =>
			MINX(
				TOPN(
					1,
					ADDCOLUMNS(VALUES(col), "Freq", CALCULATE(COUNTROWS(tab))),
					[Freq], DESC
				),
				col
			)
```

Usage: `Mode('Sales', 'Sales'[ProductKey])`

**Why AnyRef:** `VALUES(col)` needs the column reference unevaluated.
`CALCULATE(COUNTROWS(tab))` needs the table reference. Val mode would
evaluate them first, breaking the logic.

### 3. PriorYearValue — Expr + AnyRef

Evaluate any measure in the prior year. The canonical time-intelligence UDF pattern:

```tmdl
createOrReplace

	function 'PriorYearValue' =
			( expression : Scalar Variant Expr, dateColumn : AnyRef ) =>
			CALCULATE(expression, SAMEPERIODLASTYEAR(dateColumn))
```

Usage: `PriorYearValue([Total Amount], 'Calendar'[Date])`

**Why Expr:** The measure must be re-evaluated inside the date-shifted CALCULATE context.
**Why AnyRef:** SAMEPERIODLASTYEAR requires the column reference unevaluated.

### 4. TodayAsDate — Parameterless Table Return

Returns today's date as a filter-compatible table via TREATAS:

```tmdl
createOrReplace

	function 'TodayAsDate' =
			() =>
			TREATAS({ TODAY() }, 'Date'[Date])
```

Usage: `CALCULATE([Total Amount], TodayAsDate())`

### 5. SplitString — VAR + Table Return

Split a string by delimiter, return one-column table:

```tmdl
createOrReplace

	function 'SplitString' =
			( s : Scalar String, delimiter : Scalar String ) =>
			VAR str = SUBSTITUTE(s, delimiter, "|")
			VAR len = PATHLENGTH(str)
			RETURN
				SELECTCOLUMNS(
					GENERATESERIES(1, len),
					"Value", PATHITEM(str, [Value], TEXT)
				)
```

Usage: `SplitString("apple,banana,cherry", ",")` → table with 3 rows.

---

## Best Practices

1. **Always specify types and subtypes** — self-documents the function, catches errors early.
   ```
   ✓ (amount : Scalar Decimal, rate : Scalar Double)
   ✗ (amount, rate)
   ```

2. **Choose the right parameter mode:**
   - `Val` — value should be pre-computed before entering the function.
   - `Expr` — expression must be re-evaluated in the function's filter context.

3. **Use `AnyRef` for model references** — columns, tables, measures — especially
   with: `VALUES`, `CALCULATE`, `TREATAS`, `SAMEPERIODLASTYEAR`, `FILTER`.

4. **Use `VAR` for readability** — break complex logic into named steps.

5. **Test edge cases** — BLANK values, empty tables, NULL strings.

6. **Keep functions focused** — one purpose, one return type.

---

## Related References

- **tmdl-standards/SKILL.md** — TMDL syntax ground rules: indentation, createOrReplace wrapper, property order
- **measures/SKILL.md** — Measures that call UDFs; when to use a UDF vs inline DAX in a measure
- **calc-groups/SKILL.md** — UDFs used inside calculationItem expressions for shared transformation logic
- **svg-measures/SKILL.md** — UDFs for SVG string assembly (e.g. shared path builder, color picker functions)

---

**Version:** 2.0
**Last Updated:** 2026-05-02
