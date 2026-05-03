---
name: calc-groups
description: >
  Reference for Power BI Calculation Groups in TMDL. Use this whenever a user
  asks for time intelligence patterns (YTD, QTD, rolling periods, prior year),
  scenario switching (Actual/Budget/Forecast), currency conversion, or any pattern
  where the same transformation needs to apply dynamically across many measures.
  Always consult this before writing individual YTD/PY/Rolling measures from scratch
  — a calc group is the correct architecture and eliminates boilerplate at scale.
version: 1.5
---

# Calculation Groups — Power BI / TMDL Reference

Calculation groups intercept `SELECTEDMEASURE()` and transform it, applying the
same logic across every measure in the model without duplicating DAX.

---

## TMDL Structure

`calculationItem` entries are nested inside `calculationGroup` at **3 TABs**.
`column` entries are siblings of `calculationGroup` at **2 TABs**.

```
createOrReplace              ← root
	table 'Name'             ← 1 TAB
		calculationGroup     ← 2 TABs
			precedence: …    ← 3 TABs
			calculationItem 'Name' = DAX_inline    ← 3 TABs (single-line)
			calculationItem 'Name' =               ← 3 TABs (multi-line)
					DAX body                       ← 5 TABs (jumps TWO levels)
				ordinal: …                         ← 4 TABs
		column Period        ← 2 TABs (sibling of calculationGroup)
			dataType: …      ← 3 TABs
		column Ordinal       ← 2 TABs
			dataType: …      ← 3 TABs
```

**Key rules:**
- Multi-line `calculationItem` uses `=` on the declaration line — **no `expression:` property**
- DAX body jumps TWO levels deeper (3 TABs → 5 TABs), same as measures
- Properties (`ordinal:`) sit at 4 TABs — one level below the declaration
- `sourceColumn: Name` — no brackets
- Always include a separate `Ordinal` column with `sortByColumn: Ordinal` on the Name column

---

## Time Intelligence Calculation Group

```tmdl
createOrReplace

	table 'Time Intelligence'

		calculationGroup
			precedence: 10

			calculationItem 'None' = SELECTEDMEASURE()

			calculationItem 'Current' = SELECTEDMEASURE()

			calculationItem 'YTD' =
					CALCULATE(
						SELECTEDMEASURE(),
						DATESYTD('Date'[Date])
					)
				ordinal: 2

			calculationItem 'QTD' =
					CALCULATE(
						SELECTEDMEASURE(),
						DATESQTD('Date'[Date])
					)
				ordinal: 3

			calculationItem 'MTD' =
					CALCULATE(
						SELECTEDMEASURE(),
						DATESMTD('Date'[Date])
					)
				ordinal: 4

			calculationItem 'PY' =
					CALCULATE(
						SELECTEDMEASURE(),
						SAMEPERIODLASTYEAR('Date'[Date])
					)
				ordinal: 5

			calculationItem 'PY YTD' =
					CALCULATE(
						SELECTEDMEASURE(),
						SAMEPERIODLASTYEAR(DATESYTD('Date'[Date]))
					)
				ordinal: 6

			calculationItem 'vs PY' =
					SELECTEDMEASURE()
					- CALCULATE(
						SELECTEDMEASURE(),
						SAMEPERIODLASTYEAR('Date'[Date])
					)
				ordinal: 7

			calculationItem 'vs PY %' =
					VAR CurrentValue = SELECTEDMEASURE()
					VAR PreviousYearValue =
						CALCULATE(
							SELECTEDMEASURE(),
							SAMEPERIODLASTYEAR('Date'[Date])
						)
					VAR DiffValue = CurrentValue - PreviousYearValue
					RETURN DIVIDE(DiffValue, PreviousYearValue)
				ordinal: 8

			calculationItem 'Rolling 3M' =
					CALCULATE(
						SELECTEDMEASURE(),
						DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -3, MONTH)
					)
				ordinal: 9

			calculationItem 'Rolling 12M' =
					CALCULATE(
						SELECTEDMEASURE(),
						DATESINPERIOD('Date'[Date], LASTDATE('Date'[Date]), -12, MONTH)
					)
				ordinal: 10

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

> **LASTDATE vs TODAY():** Always anchor rolling windows to `LASTDATE('Date'[Date])`,
> not `TODAY()`. LASTDATE respects the visual's date filter; TODAY() ignores it.

---

## Scenario / Budget Calculation Group

```tmdl
createOrReplace

	table Scenario

		calculationGroup
			precedence: 20

			calculationItem 'None' = SELECTEDMEASURE()

			calculationItem 'Actual' =
					CALCULATE(
						SELECTEDMEASURE(),
						FactTable[Scenario] = "Actual"
					)
				ordinal: 1

			calculationItem 'Budget' =
					CALCULATE(
						SELECTEDMEASURE(),
						FactTable[Scenario] = "Budget"
					)
				ordinal: 2

			calculationItem 'Forecast' =
					CALCULATE(
						SELECTEDMEASURE(),
						FactTable[Scenario] = "Forecast"
					)
				ordinal: 3

			calculationItem 'vs Budget' =
					VAR _actual =
						CALCULATE(SELECTEDMEASURE(), FactTable[Scenario] = "Actual")
					VAR _budget =
						CALCULATE(SELECTEDMEASURE(), FactTable[Scenario] = "Budget")
					RETURN _actual - _budget
				ordinal: 4

			calculationItem 'vs Budget %' =
					VAR _actual =
						CALCULATE(SELECTEDMEASURE(), FactTable[Scenario] = "Actual")
					VAR _budget =
						CALCULATE(SELECTEDMEASURE(), FactTable[Scenario] = "Budget")
					RETURN DIVIDE(_actual - _budget, _budget)
				ordinal: 5

		column Scenario
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

---

## Currency Conversion Calculation Group

```tmdl
createOrReplace

	table Currency

		calculationGroup
			precedence: 5

			calculationItem 'USD' = SELECTEDMEASURE()

			calculationItem 'EUR' =
					VAR _rate =
						CALCULATE(
							AVERAGE(ExchangeRates[Rate]),
							ExchangeRates[CurrencyCode] = "EUR"
						)
					RETURN SELECTEDMEASURE() * _rate
				ordinal: 2

			calculationItem 'GBP' =
					VAR _rate =
						CALCULATE(
							AVERAGE(ExchangeRates[Rate]),
							ExchangeRates[CurrencyCode] = "GBP"
						)
					RETURN SELECTEDMEASURE() * _rate
				ordinal: 3

			calculationItem 'JPY' =
					VAR _rate =
						CALCULATE(
							AVERAGE(ExchangeRates[Rate]),
							ExchangeRates[CurrencyCode] = "JPY"
						)
					RETURN SELECTEDMEASURE() * _rate
				ordinal: 4

		column Currency
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

---

## Precedence

**Higher number = applied first (outermost).** Lower number wraps last.

```
Precedence 20 → Scenario  (outermost)
Precedence 10 → Time      (middle)
Precedence  5 → Currency  (innermost)
```

## Excluding Specific Measures

```tmdl
			calculationItem 'YTD' =
					IF(
						ISSELECTEDMEASURE([# Headcount], [Balance]),
						SELECTEDMEASURE(),
						CALCULATE(SELECTEDMEASURE(), DATESYTD('Date'[Date]))
					)
				ordinal: 2
```

---

## Checklist Before Publishing

- [ ] `calculationItem` at 3 TABs — nested inside `calculationGroup`
- [ ] Multi-line items use `=` on the declaration line, DAX at 5 TABs, properties at 4 TABs
- [ ] No `expression:` property, no `formatStringDefinition`, no `formatStringExpression`
- [ ] `column` and `column Ordinal` at 2 TABs — siblings of `calculationGroup`
- [ ] `sourceColumn: Name` (no brackets), `sortByColumn: Ordinal`
- [ ] Every group has a `None` item (`= SELECTEDMEASURE()`)
- [ ] Rolling window items use `LASTDATE('Date'[Date])`, not `TODAY()`
- [ ] Precedence order intentional: 20 → Scenario, 10 → Time, 5 → Currency

---

## Related References

- **tmdl-standards/SKILL.md** — TMDL syntax ground rules: indentation, createOrReplace wrapper, property order
- **measures/SKILL.md** — Base measures that calc group items operate on; naming and folder conventions
- **dax-udf/SKILL.md** — UDFs that can be called from within calculationItem expressions
- **svg-measures/SKILL.md** — SVG icon badges reflecting the active calc group selection (e.g. period label)

---

**Version:** 1.5
**Last Updated:** 2026-05-02
