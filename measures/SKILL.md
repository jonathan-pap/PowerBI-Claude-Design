---
name: measures
description: Comprehensive guide to designing, naming, and implementing Power BI DAX measures across all business domains. Includes patterns for base aggregations, time intelligence, KPIs, semi-additive measures, and TMDL syntax for measure definitions in .tmdl files.
version: 2.0
---

# Power BI DAX Measures — Design Studio Guide

This guide teaches how to scaffold a complete measure layer for any Power BI semantic model. Use this when a user describes their reporting needs to generate production-ready DAX expressions with proper naming, folder structure, and TMDL syntax.

---

## 1. Naming Conventions & Prefixes

All measures use **prefixes** to signal their type.

| Prefix | Type | Example | formatString |
|--------|------|---------|--------------|
| `#` | Count / Distinct Count | `# Orders`, `# Customers` | `#,0` |
| `$` | Currency | `$ Revenue`, `$ Cost` | `"$#,0.00"` |
| `%` | Ratio / Percentage | `% Margin`, `% Growth` | `"0.00%"` |
| `Δ` | Variance (Absolute) | `Δ Revenue`, `Δ Headcount` | `"#,0"` or `"$#,0"` |
| `Δ%` | Variance (Percent) | `Δ% Revenue`, `Δ% Target` | `"0.00%"` |
| (none) | Helper / Dimension | `Days in Period`, `Current Month Num` | varies |

### Naming Principles

- **Verbose over cryptic**: `$ Revenue Net of Discounts` beats `$ Rev ND`
- **Avoid abbreviations** except standard business terms (YTD, QTD, MTD, PY)
- **Order variants left-to-right**: base first, then comparisons
  - ✓ `$ Revenue`, `$ Revenue PY`, `$ Revenue YTD`
  - ✗ `$ PY Revenue`, `$ YTD Revenue`

---

## 2. Display Folder Structure

```
├── Key Metrics
│   ├── Revenue & Profitability
│   ├── Efficiency
│   └── Quality
├── Time Intelligence
│   ├── Year-to-Date
│   ├── Prior Year Comparisons
│   └── Period-over-Period
├── Budget vs Actual
│   ├── Actual Measures
│   ├── Budget Measures
│   └── Variance
├── Ratios & Rates
└── Drilling & Details
    └── Helper Calculations
```

**Rules:**
- One or two levels deep — avoid deeper nesting
- Align folders with report pages: a "Variance Analysis" page → `Variance Analysis` folder
- Group by business metric, not by function (`Key Metrics`, not `SUM Measures`)

---

## 3. TMDL Wrapper

All generated TMDL uses `createOrReplace` targeting `_measures`.
Measures sit at **2 TABs**, properties at **3 TABs**, multi-line DAX body at **4 TABs**:

```tmdl
createOrReplace

	ref table _measures

		/// Description of the measure
		measure '$ Revenue' = SUM(Sales[Revenue])
			formatString: "$#,0.00"
			displayFolder: "Key Metrics\Revenue & Profitability"
```

The `createOrReplace` / `ref table _measures` header is omitted from the pattern
examples below for brevity — always include it in generated output.

---

## 4. Base Measure Patterns

### Pattern 1: Simple Sum / Count

**DAX:**
```dax
$ Revenue = SUM(Sales[Revenue])
# Orders = COUNTROWS(Sales)
```

**TMDL:**
```tmdl
		/// Total revenue from all sales transactions
		measure '$ Revenue' = SUM(Sales[Revenue])
			formatString: "$#,0.00"
			displayFolder: "Key Metrics\Revenue & Profitability"

		/// Number of sales orders
		measure '# Orders' = COUNTROWS(Sales)
			formatString: "#,0"
			displayFolder: "Key Metrics"
```

### Pattern 2: SUMX for Row-Level Calculation

**DAX:**
```dax
$ Revenue = SUMX(Sales, Sales[Quantity] * Sales[Unit Price])
```

**TMDL:**
```tmdl
		measure '$ Revenue' =
				SUMX(Sales, Sales[Quantity] * Sales[Unit Price])
			formatString: "$#,0.00"
			displayFolder: "Key Metrics\Revenue & Profitability"
```

### Pattern 3: Conditional Aggregation with CALCULATE + FILTER

**DAX:**
```dax
# High-Value Orders =
CALCULATE(
    COUNTROWS(Sales),
    Sales[Revenue] > 1000
)
```

**TMDL:**
```tmdl
		measure '# High-Value Orders' =
				CALCULATE(
					COUNTROWS(Sales),
					Sales[Revenue] > 1000
				)
			formatString: "#,0"
			displayFolder: "Key Metrics"
```

### Pattern 4: Distinct Count

**DAX:**
```dax
# Customers = DISTINCTCOUNT(Sales[CustomerID])
```

**TMDL:**
```tmdl
		measure '# Customers' = DISTINCTCOUNT(Sales[CustomerID])
			formatString: "#,0"
			displayFolder: "Key Metrics"
```

### Pattern 5: Average / Rate with DIVIDE

**DAX:**
```dax
$ Average Order Value =
DIVIDE(
    SUM(Sales[Revenue]),
    DISTINCTCOUNT(Sales[OrderID]),
    0
)
```

**TMDL:**
```tmdl
		measure '$ Average Order Value' =
				DIVIDE(
					SUM(Sales[Revenue]),
					DISTINCTCOUNT(Sales[OrderID]),
					0
				)
			formatString: "$#,0.00"
			displayFolder: "Key Metrics\Efficiency"
```

---

## 5. Time Intelligence Patterns

> Requires a marked Date table with a `[Date]` column and an active relationship to fact tables.

### Pattern 1: Year-to-Date (YTD)

**DAX:**
```dax
$ Revenue YTD =
CALCULATE(
    [$ Revenue],
    DATESYTD(Date[Date])
)
```

**TMDL:**
```tmdl
		measure '$ Revenue YTD' =
				CALCULATE(
					[$ Revenue],
					DATESYTD(Date[Date])
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Year-to-Date"
```

### Pattern 2: Quarter-to-Date (QTD)

**DAX:**
```dax
$ Revenue QTD =
CALCULATE(
    [$ Revenue],
    DATESQTD(Date[Date])
)
```

**TMDL:**
```tmdl
		measure '$ Revenue QTD' =
				CALCULATE(
					[$ Revenue],
					DATESQTD(Date[Date])
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Quarter-to-Date"
```

### Pattern 3: Month-to-Date (MTD)

**DAX:**
```dax
$ Revenue MTD =
CALCULATE(
    [$ Revenue],
    DATESMTD(Date[Date])
)
```

**TMDL:**
```tmdl
		measure '$ Revenue MTD' =
				CALCULATE(
					[$ Revenue],
					DATESMTD(Date[Date])
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Month-to-Date"
```

### Pattern 4: Prior Year (PY)

**DAX:**
```dax
$ Revenue PY =
CALCULATE(
    [$ Revenue],
    SAMEPERIODLASTYEAR(Date[Date])
)
```

**TMDL:**
```tmdl
		measure '$ Revenue PY' =
				CALCULATE(
					[$ Revenue],
					SAMEPERIODLASTYEAR(Date[Date])
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Prior Year Comparisons"
```

### Pattern 5: Prior Year YTD

**DAX:**
```dax
$ Revenue PY YTD =
CALCULATE(
    [$ Revenue],
    SAMEPERIODLASTYEAR(DATESYTD(Date[Date]))
)
```

**TMDL:**
```tmdl
		measure '$ Revenue PY YTD' =
				CALCULATE(
					[$ Revenue],
					SAMEPERIODLASTYEAR(DATESYTD(Date[Date]))
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Prior Year Comparisons"
```

### Pattern 6: Rolling N Months

**DAX:**
```dax
$ Revenue L12M =
CALCULATE(
    [$ Revenue],
    DATESINPERIOD(Date[Date], LASTDATE(Date[Date]), -12, MONTH)
)
```

**TMDL:**
```tmdl
		measure '$ Revenue L12M' =
				CALCULATE(
					[$ Revenue],
					DATESINPERIOD(Date[Date], LASTDATE(Date[Date]), -12, MONTH)
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Period-over-Period"
```

> Use `LASTDATE(Date[Date])` not `TODAY()` — LASTDATE respects the visual's date filter.

---

## 6. KPI Measures — Variance, Status, Targets

### Pattern 1: Absolute Variance

**DAX:**
```dax
Δ Revenue vs Target = [$ Revenue] - [Target Revenue]
```

**TMDL:**
```tmdl
		measure 'Δ Revenue vs Target' = [$ Revenue] - [Target Revenue]
			formatString: "$#,0.00"
			displayFolder: "Budget vs Actual\Variance"
```

### Pattern 2: Variance as Percentage

**DAX:**
```dax
Δ% Revenue vs Target =
DIVIDE(
    [$ Revenue] - [Target Revenue],
    [Target Revenue],
    0
)
```

**TMDL:**
```tmdl
		measure 'Δ% Revenue vs Target' =
				DIVIDE(
					[$ Revenue] - [Target Revenue],
					[Target Revenue],
					0
				)
			formatString: "0.00%"
			displayFolder: "Budget vs Actual\Variance"
```

### Pattern 3: Status Flag (−1 / 0 / 1)

Returns `1` (above target), `0` (no data), `−1` (below target) — drives conditional formatting or SVG icons.

**DAX:**
```dax
Status Revenue =
IF(
    ISBLANK([$ Revenue]),
    0,
    IF([$ Revenue] >= [Target Revenue], 1, -1)
)
```

**TMDL:**
```tmdl
		measure 'Status Revenue' =
				IF(
					ISBLANK([$ Revenue]),
					0,
					IF([$ Revenue] >= [Target Revenue], 1, -1)
				)
			formatString: "0"
			displayFolder: "Budget vs Actual"
```

### Pattern 4: Target / Budget Measure

**DAX:**
```dax
Target Revenue = SUM(Budget[TargetRevenue])
```

**TMDL:**
```tmdl
		measure 'Target Revenue' = SUM(Budget[TargetRevenue])
			formatString: "$#,0.00"
			displayFolder: "Budget vs Actual\Budget Measures"
```

---

## 7. Semi-Additive Measures

Semi-additive measures must not be summed across time. Use `LASTNONBLANK` or `LASTDATE`.

### Pattern 1: Period-End Balance (LASTNONBLANK)

**Use case:** Headcount, inventory, account balance at end of period.

**DAX:**
```dax
# Headcount EOP =
CALCULATE(
    SUM(Headcount[Employees]),
    LASTNONBLANK(Date[Date], CALCULATE(SUM(Headcount[Employees])))
)
```

**TMDL:**
```tmdl
		measure '# Headcount EOP' =
				CALCULATE(
					SUM(Headcount[Employees]),
					LASTNONBLANK(Date[Date], CALCULATE(SUM(Headcount[Employees])))
				)
			formatString: "#,0"
			displayFolder: "Key Metrics"
```

### Pattern 2: Period-End Balance (LASTDATE)

**DAX:**
```dax
$ Account Balance EOD =
CALCULATE(
    SUM(GL[Balance]),
    LASTDATE(Date[Date])
)
```

**TMDL:**
```tmdl
		measure '$ Account Balance EOD' =
				CALCULATE(
					SUM(GL[Balance]),
					LASTDATE(Date[Date])
				)
			formatString: "$#,0.00"
			displayFolder: "Key Metrics"
```

### Pattern 3: Average of Semi-Additive

**DAX:**
```dax
$ Average Daily Balance =
AVERAGEX(
    VALUES(Date[Date]),
    [$ Account Balance EOD]
)
```

**TMDL:**
```tmdl
		measure '$ Average Daily Balance' =
				AVERAGEX(
					VALUES(Date[Date]),
					[$ Account Balance EOD]
				)
			formatString: "$#,0.00"
			displayFolder: "Key Metrics"
```

---

## 8. Error Handling

### DIVIDE (preferred over `/`)

Always use `DIVIDE(numerator, denominator, alternateResult)` — returns `alternateResult` (default `BLANK()`) instead of an error when denominator is zero.

```dax
% Margin =
DIVIDE(
    SUM(Sales[Profit]),
    SUM(Sales[Revenue]),
    0
)
```

### IFERROR

Catches any error in an expression:

```dax
% Utilization =
IFERROR(
    DIVIDE(SUM(Hours[Billable]), SUM(Hours[Total]), BLANK()),
    BLANK()
)
```

### VAR + ISBLANK guard

Prevents downstream errors when a dependency measure is blank:

```dax
$ Revenue vs Target =
VAR _actual = [$ Revenue]
VAR _target = [Target Revenue]
RETURN
    IF(
        ISBLANK(_target),
        BLANK(),
        _actual - _target
    )
```

---

## 9. formatString Quick Reference

| Pattern | Displays as | Use for |
|---------|-------------|---------|
| `"$#,0.00"` | $1,234.56 | USD currency |
| `"€#,0.00"` | €1,234.56 | EUR currency |
| `"#,0"` | 1,235 | Whole number counts |
| `"0.0%"` | 12.3% | Percentages (1 decimal) |
| `"0.00%"` | 12.35% | Percentages (2 decimal) |
| `"#,0,,"M"` | 1.2M | Compact millions |
| `"#,0,"K"` | 1.2K | Compact thousands |

---

## 10. Measure Assembly — Requirements to TMDL

When a user describes reporting needs, follow this process:

**Example:** "Track quarterly sales with prior-year comparison and variance to quota."

**Step 1 — Extract metrics:**
- `$ Revenue` — base sum
- `$ Revenue PY` — prior year comparison
- `Target Quota` — budget/target
- `Δ Revenue vs Quota` — absolute variance
- `Δ% Revenue vs Quota` — percent variance

**Step 2 — Assign folders:**
```
Key Metrics\Revenue & Profitability  → $ Revenue
Budget vs Actual\Budget Measures     → Target Quota
Budget vs Actual\Variance            → Δ Revenue vs Quota, Δ% Revenue vs Quota
Time Intelligence\Prior Year         → $ Revenue PY
```

**Step 3 — Generate TMDL:**

```tmdl
createOrReplace

	ref table _measures

		measure '$ Revenue' = SUM(Sales[Amount])
			formatString: "$#,0.00"
			displayFolder: "Key Metrics\Revenue & Profitability"

		measure 'Target Quota' = SUM(Budget[QuotaAmount])
			formatString: "$#,0.00"
			displayFolder: "Budget vs Actual\Budget Measures"

		measure '$ Revenue PY' =
				CALCULATE(
					[$ Revenue],
					SAMEPERIODLASTYEAR(Date[Date])
				)
			formatString: "$#,0.00"
			displayFolder: "Time Intelligence\Prior Year Comparisons"

		measure 'Δ Revenue vs Quota' = [$ Revenue] - [Target Quota]
			formatString: "$#,0.00"
			displayFolder: "Budget vs Actual\Variance"

		measure 'Δ% Revenue vs Quota' =
				DIVIDE([Δ Revenue vs Quota], [Target Quota], 0)
			formatString: "0.00%"
			displayFolder: "Budget vs Actual\Variance"
```

---

## 11. Key Properties Reference

| Property | Purpose | Example |
|----------|---------|---------|
| `formatString` | Display format in visuals | `"$#,0.00"`, `"0.00%"`, `"#,0"` |
| `displayFolder` | Group in Field List | `"Key Metrics\Revenue"` |
| `description` | Tooltip in Field List | `"Cumulative revenue YTD"` |
| `isHidden` | Hide from Field List | `isHidden` (for helper measures) |
