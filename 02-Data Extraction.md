# Data extraction

## Required data and where to find it

| What | Where to look | Key columns |
|---|---|---|
| Budget by product | Sheet with `Budget` in the name, or a `Period` column equal to `Budget` | Product ID, Category, Month, Units, DM Cost, DL Cost, VOH Cost |
| Actual volumes | Sheet with `Volumes` in the name | Product ID, Units Produced, Units Sold, Ending Inventory |
| Actual COGS totals | GL, P&L, or Income Statement sheet | DM, DL, VOH total actual COGS |
| Allocated actuals | Derived from Production Log + GL | Per-product actual DM, DL, VOH COGS on a units-sold basis |
| Production Log | Sheet with `Production` in the name, if allocation is needed | Work orders by Product ID with material quantities, labor hours |

## Flexible sheet matching

Do not hardcode sheet names. Scan sheet names for keywords: `Budget`, `Actual`, `Product`, `Volume`, `GL`, `P&L`, `Production Log`, `Production`.

Read row 1 as the header row and map columns by header text, not position. Header matching should tolerate normal variation in capitalization, spacing, punctuation, and common naming differences (e.g. "Product ID" vs "ProductID" vs "SKU"). Never map a column solely because it happens to sit in a particular position.

## Month filtering

Budget sheets often contain all 12 months in one sheet.

**Single month:** filter budget data to the requested month, filter actual data to the same month, use that period consistently across all three cost lines.

**YTD:** aggregate budget data from January through the requested month, aggregate actual data across the same months using only months that have actual data, run the same PVM calculation on the aggregated totals. Commentary should reference cumulative trends, not a single month's drivers. If the data supports it, also show a month-by-month trend of the key variances. Don't treat each month as an independent full comparison — aggregate the underlying units and costs first, then calculate PVM once on the totals.

## Critical: standard costs disguised as actual costs

The PVM analysis needs **true actual COGS** per product on a units-sold basis. Many workbooks have a `Product Actuals` sheet that looks like actual costs but actually applies standard material prices to actual quantities. If this gets used as though it were true actual cost, the Price variance comes out understated — sometimes at or near zero when a real price change occurred.

**How to check:** compare per-unit material costs in the actuals sheet against the budget rates for the same products. If actual and budget rates are identical across every product despite real-world purchase-price movement you'd expect to see, the sheet is standard-cost-based, not true actual. Do this check before using any product-level actual cost data in the Price calculation.

## Building true actual cost: two paths

### Path A — GL totals + Production Log available

Use this when the GL provides actual DM, DL, and VOH COGS totals, and the Production Log provides the product-level activity needed to allocate those totals down to individual products.

**Direct Materials.** Compute each product's material cost at standard prices from the Production Log. Each product's share of standard DM cost:

Product DM Share = Product Standard DM Cost ÷ Total Standard DM Cost

Allocate the GL's actual total:

Product Actual DM COGS = Product DM Share × GL Actual DM COGS

**Direct Labor.** Use labor hours as the allocation base. Each product's labor-hour share:

Product Labor Share = Product Labor Hours ÷ Total Labor Hours

Allocate the GL's actual total:

Product Actual DL COGS = Product Labor Share × GL Actual DL COGS

**Variable Overhead.** Use labor hours as the allocation base unless the workbook provides a more appropriate VOH base (machine hours and production volume are the next most common — check for a sheet or column that explicitly ties VOH to a different driver before defaulting to labor hours). Same share calculation as Direct Labor:

Product Actual VOH COGS = Product Labor Share × GL Actual VOH COGS

All three results are on a units-sold basis automatically, because the GL records COGS for units sold, not units produced.

### Path B — Product Actuals only, no GL/Production Log

First confirm (using the check above) that the Product Actuals sheet is genuinely true actual cost, not standard prices relabeled.

If it contains true actual total cost based on production:

Actual $/unit = Actual Total Cost ÷ Units Produced

Then convert to a COGS (units-sold) basis:

Actual COGS = Actual $/unit × Units Sold

Use this Actual COGS in the PVM analysis — this conversion is required because PVM compares cost associated with units sold, and a production-based total cost isn't directly comparable without it.

If the Product Actuals sheet turns out to be standard-price-based, don't treat it as true actual data. Use Path A if the GL and Production Log are available, or ask the user for actual purchase-price or GL data.

## Data validation checklist

Before calculating anything, verify:

1. Product IDs match between budget and actuals.
2. Units sold are available, not only units produced.
3. Cost columns contain numeric values.
4. Actual costs are confirmed on a units-sold basis (via allocation, Path A, or explicit conversion, Path B).
5. If using Path A, GL COGS totals reconcile to the sum of allocated per-product actual costs.
6. Budget units and actual units are aligned to the same period.
7. Each cost line has a valid budget amount and actual amount.
8. Unit cost calculations never divide by zero.
9. Product categories are mapped consistently.
10. Duplicate product-period records are identified and resolved before aggregating.
11. Negative quantities or costs are investigated, not silently accepted.
12. Sum of product-level budget costs reconciles to the relevant budget total, where one exists.
13. Sum of product-level actual costs reconciles to the relevant actual total.
14. The full PVM bridge reconciles to actual COGS minus budget COGS (see reconciliation tolerance in `pvm-calculations.md`).

If a product has no actual record at all, include it with actual units and actual COGS set to zero rather than dropping it.

## Units produced vs. units sold

- Units produced matters only when converting a production-based actual cost into a per-unit figure (Path B).
- Units sold is the volume basis for COGS and every PVM calculation.
- Ending inventory helps explain the gap between production and sales when they diverge.
- Never use units produced as the PVM volume unless the underlying COGS data is also explicitly production-based and has been converted per Path B.

When production exceeds sales, check whether the resulting inventory build explains why production-based costs differ from COGS.

## Zero-unit and new/discontinued products

If budget units are zero, don't calculate a Budget $/unit by dividing by zero. If actual units are zero, don't calculate an Actual $/unit by dividing by zero. Instead:

- Flag the product as a zero-volume case.
- Determine whether it's new, discontinued, inactive, or just missing data.
- Let its contribution flow through the aggregate PVM methodology (the mix framework) rather than forcing a per-unit calculation.
- Note the treatment in verification notes if the amount is material.

**Budget units = 0, actual units > 0:** treat as a product introduced during the period.
**Budget units > 0, actual units = 0:** treat as a product absent or discontinued during the period.

In both cases, use the mix framework to capture the product's contribution to the change in composition rather than computing a conventional unit-price variance with a zero denominator.
