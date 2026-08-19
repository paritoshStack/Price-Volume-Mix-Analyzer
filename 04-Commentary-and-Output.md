# Commentary and workbook output

## Deliverable

The output is **the source workbook, updated in place** with new sheets added (or a new copy of it, if the user prefers to keep the original untouched — ask if unclear). Don't produce a separate chat-only report as the primary deliverable; the workbook is what gets shared with management.

## Sheet structure

Add these sheets, in this order, after the existing data sheets:

1. **Volume Data** — the shared budget-vs-actual units-sold table from calculation step 1.
2. **DM Analysis** — per-product PVM table (both methods) plus the aggregate three-way bridge, for Direct Materials.
3. **DL Analysis** — same, for Direct Labor.
4. **VOH Analysis** — same, for Variable Overhead.
5. **Summary Bridge** — the consolidated nine-step waterfall (see below) across all three lines, with the chart.
6. **Verification** — every reconciliation check from this file, with pass/fail and the tolerance applied.
7. **Commentary** — the written variance commentary for all three lines.

## Per-product PVM table (each cost-line sheet)

Include at least these columns, showing **both** decomposition methods side by side:

| Product | Category | Budget Qty | Actual Qty | Δ Volume | Budget $/Unit | Actual $/Unit | Two-Step Volume | Two-Step Price | Three-Way Volume | Three-Way Mix | Three-Way Price | Total Variance |
|---|---|---|---|---|---|---|---|---|---|---|---|---|

The Total Variance column should be identical whether you sum the two-step columns or the three-way columns for that row — that's a row-level version of the same reconciliation check used at the aggregate level in `pvm-calculations.md`.

The per-product Mix formula is the same one used at the aggregate level:

Mix Variance_i = (Actual Units_i − (Actual Total Units × Budget Mix Share_i)) × Budget Unit Cost_i

Repeat this here rather than only defining it once elsewhere — it's the formula most likely to get applied inconsistently between the per-product table and the aggregate bridge if it's only stated once.

## Aggregate bridge (each cost-line sheet, and again in Summary Bridge)

Show both totals clearly labeled:

- **Two-step total:** Budget COGS → Volume → Price → Actual COGS
- **Three-way total:** Budget COGS → Volume → Mix → Price → Actual COGS

Both should equal the same Actual COGS figure. Show the cross-method identity check explicitly (Two-step Volume = Three-way Volume + Three-way Mix) with a pass/fail flag using the tolerance from `pvm-calculations.md`.

## Summary Bridge sheet

The nine-step consolidated bridge, across all three cost lines, using the three-way decomposition (the two-step figures live on the individual cost-line sheets and in the identity check, but the consolidated cross-line bridge uses three-way, since that's the version that isolates mix):

Budget Total → DM Volume → DM Mix → DM Price → DL Volume → DL Mix → DL Price → VOH Volume → VOH Mix → VOH Price → Actual Total

Must reconcile:

Budget Total + all nine steps = Actual Total

## Waterfall chart

Build an 11-bar stacked-column chart simulating a waterfall: Budget Total, the nine variance steps above, Actual Total.

The specific charting mechanism depends on the tool available. If using a library that doesn't support native waterfall or floating-bar charts (for example, ExcelJS in Node), one workable approach is building the chart from stacked-column series with an invisible "base" series to create the floating-bar effect, injecting the chart via direct OOXML manipulation (JSZip) if the library has no chart API at all — with helper data for the chart placed in a clearly labeled block of helper columns on the Summary Bridge sheet, documented with a header row so the layout isn't a silent assumption. If using a Python toolchain (openpyxl, xlsxwriter), those have more native chart support and may not need this workaround at all. Pick whichever approach the available tooling actually supports — the requirement is the visual result (an 11-bar waterfall bridging budget to actual), not a specific implementation technique.

Anchor the chart on the Summary Bridge sheet, above the bridge table.

Color coding: blue for the Budget Total and Actual Total bars, red for unfavorable (cost-increasing) steps, green for favorable (cost-decreasing) steps.

## Verification sheet

Include every check below, each with the computed values, the tolerance applied, and a clear pass/fail flag.

**Budget reconciliation:** sum of product budget costs reconciles to the relevant budget total, where one exists.

**Actual reconciliation:** sum of product actual costs reconciles to the relevant actual GL total. For Path A specifically: allocated product actual DM/DL/VOH COGS should each equal the corresponding GL actual total.

**Total variance reconciliation**, per cost line: Actual COGS − Budget COGS = Total PVM Variance, and Total PVM Variance = Volume + Mix + Price (three-way) = Volume + Price (two-step).

**Cross-method identity**, per cost line: Two-step Volume = Three-way Volume + Three-way Mix, and Two-step Price = Three-way Price.

**Full bridge reconciliation**, across all three lines: Actual Total COGS − Budget Total COGS = sum of all nine three-way steps.

**Product mix check:** Budget Mix Share_i = Budget Units_i ÷ Total Budget Units; Actual Mix Share_i = Actual Units_i ÷ Total Actual Units; Mix Change_i = Actual Mix Share_i − Budget Mix Share_i. Use this to name which products or categories drove the largest mix shifts.

**Volume check:** Σ Product Δ Volume = Actual Total Units − Budget Total Units.

**Category check:** aggregate PVM by product category, where categories exist, to see whether the mix effect is concentrated in particular categories (e.g. Premium vs. Basic).

**Inventory check**, where Units Produced and Ending Inventory exist: Ending Inventory Change = Actual Units Produced − Actual Units Sold. Use as a diagnostic when production and sales volumes diverge materially. Don't automatically fold inventory changes into COGS PVM unless the accounting basis requires it.

If any check fails outside tolerance, stop and investigate before writing commentary — don't write commentary around numbers that haven't reconciled.

## Commentary tone and content

Write as a finance professional presenting to management: specific with numbers, but don't just repeat the table. Interpret what the numbers mean for the business, and flag anything warranting attention — large unfavorable variances, large favorable ones that may not be sustainable, unusual or recurring patterns, material mix shifts, cost increases that look structural rather than temporary.

Don't claim a causal driver the workbook doesn't support. Where the data doesn't establish a cause, state the observed pattern and say what additional data would pin down the cause.

## Commentary template, per cost line

**[Cost Line] Variance Commentary**

Total Variance: $X,XXX [favorable/unfavorable] (X.X% of budget)

Volume — Two-Step: $X,XXX. Three-Way: $X,XXX. Explain what drove the unit-volume change: which products or categories grew or shrank, whether the pattern looks seasonal, operational, customer-driven, or one-time (only if the data supports the distinction).

Mix (three-way only): $X,XXX. Which categories gained or lost share, whether the mix moved toward higher- or lower-cost products, and the resulting cost impact.

Price: $X,XXX (identical under both methods). What drove the per-unit cost change — see line-specific guidance below.

## Line-specific driver guidance

**Direct Materials.** Price drivers: commodity price changes, supplier negotiations, purchase volume discounts, material substitutions. Mix insight: Premium products often use costlier materials (aluminum, copper) versus Basic products (steel, plastic only) — a shift toward Premium raises average material cost per unit even with unchanged rates. Where BOM data exists, check whether the mix variance reflects different material quantities per product, different material types, higher-cost components, or a category-level mix shift.

**Direct Labor.** Price drivers: wage rate changes, overtime premiums, skill mix of the workforce, hiring more or less experienced staff. Volume and mix interact here more than in materials: Premium products often need roughly 1.8–2.2 labor hours per unit versus 0.5–0.7 for Basic, so a mix shift toward Premium has an outsized labor-cost effect. Where data allows, compare Actual Hours/Unit against Standard Hours/Unit to separate a rate effect from a labor-efficiency effect.

**Variable Overhead.** VOH is typically driven by labor hours (VOH Cost = VOH Rate × Labor Hours), so its pattern often mirrors Direct Labor. Price variance is the change in VOH Rate per Labor Hour vs. standard. Mix effect follows the same logic as labor — more labor-intensive products absorb more VOH. Where data permits, check whether changes stem from the VOH rate itself, labor-hour changes, product mix, or the underlying variable overhead cost pool.

## Management interpretation summary

Pull together, across all three lines: the **Volume story** (units sold vs. plan, which products/categories moved most, concentrated or broad-based), the **Price story** (which line had the largest price variance, which products drove unit-cost changes, likely temporary or recurring if the evidence supports a view), and the **Mix story** (categories gaining/losing share, whether high-cost products gained or lost share, the net COGS impact).

## Number formatting

Commas for thousands. Parentheses for negatives in tables — `(994)`, never `-994`. Two decimal places for per-unit costs, whole dollars for totals and total variances, percentages to one decimal place unless the user asks for more precision. Keep units consistent throughout — don't mix a per-unit column in cents with a total column in dollars.

## Output order (Commentary sheet and any accompanying summary)

1. Title: `PVM VARIANCE ANALYSIS — COGS BY LINE — [Month] [Year]`
2. Volume data table
3. Direct Materials: per-product table (both methods) + aggregate bridge (both methods)
4. Direct Labor: same
5. Variable Overhead: same
6. Summary bridge (three-way) + waterfall chart
7. Verification checks, with pass/fail
8. Variance commentary: concise, 1–2 sentences per component, per cost line
