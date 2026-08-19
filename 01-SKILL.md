---
name: pvm-variance-analysis
description: Perform Price-Volume-Mix (PVM) variance analysis on standard-cost manufacturing data. Use whenever the user asks to explain why actual COGS (Direct Materials, Direct Labor, or Variable Overhead) differed from budget, wants a PVM bridge or waterfall chart, asks for a volume/price/mix breakdown of a cost or margin variance, or uploads a budget-vs-actual workbook and asks what drove the difference. Reads Excel workbooks with budget and actual figures and writes an updated workbook containing the full PVM analysis, verification checks, a waterfall chart, and management commentary.
---

# PVM variance analysis

Decomposes total COGS variance into Volume, Price, and Mix components, for each of three cost lines — Direct Materials (DM), Direct Labor (DL), Variable Overhead (VOH) — and explains the business drivers behind each one. The output is an updated Excel workbook: the original data plus new sheets containing the analysis, the reconciliation checks, the waterfall chart, and written commentary.

The core question this answers: *actual cost differed from budget — was that because we sold a different quantity (Volume), paid a different rate or unit cost (Price), or sold a different mix of products (Mix)?*

Calculate **both** decomposition methods for every cost line — the two-step (Volume + Price only) and the three-way (Volume + Mix + Price) — and show both in the final output. They tie together with a specific identity (see `references/pvm-calculations.md`), and showing both lets a reader see exactly how the mix effect was carved out of what the two-step method lumps into "Volume."

Reference files, read them as needed rather than all at once:

- `references/data-extraction.md` — finding and validating the workbook data, the two paths (GL+Production Log, or Product Actuals) for building true actual costs, handling missing/zero-unit products
- `references/pvm-calculations.md` — every formula for both decomposition methods, the reconciliation identity between them, sign conventions, a full worked example
- `references/commentary-and-output.md` — the workbook output structure, per-line commentary guidance, the waterfall chart, number formatting, and the verification checks

## Workflow

1. **Identify the data source.** Find the Excel file, usually in `docs/`. If more than one exists, ask the user which one.
2. **Identify the period.** A specific month, or YTD. If not specified, ask.
3. **Read the workbook and locate the relevant sheets.** See `references/data-extraction.md` for sheet-matching rules — do not hardcode sheet names or column positions.
4. **Validate the workbook** before calculating anything — product ID matching, units-sold availability, numeric cost columns, duplicate records. Full checklist in `references/data-extraction.md`.
5. **Establish true actual costs on a units-sold basis** for DM, DL, and VOH. This is the step most likely to go wrong — many workbooks have a sheet that looks like actual costs but is actually standard prices applied to actual quantities. Verify before using it; see `references/data-extraction.md`.
6. **Run both PVM decompositions** — two-step and three-way — independently for DM, DL, and VOH. Formulas and worked example in `references/pvm-calculations.md`.
7. **Reconcile.** Both decompositions must sum to the same total variance per line, and the two-step Volume figure must equal the three-way Volume plus Mix figures (the cross-method identity). Reconciliation tolerance and what to do if it fails: `references/pvm-calculations.md`.
8. **Write variance commentary** explaining the business drivers behind each component. Tone and line-specific guidance (materials, labor, VOH) in `references/commentary-and-output.md`.
9. **Write the updated workbook**: per-product tables (both methods), the aggregate bridge, the nine-step waterfall chart, verification checks, and commentary. Full structure in `references/commentary-and-output.md`.

## Data extraction

Read the workbook with Node.js and the `xlsx` package (or the equivalent Python approach — pandas/openpyxl — if that's the environment's convention; the logic below is tool-agnostic).

The workbook structure varies between companies. Do not assume fixed sheet names or fixed column positions — scan sheet names for keywords (`Budget`, `Actual`, `Product`, `Volume`, `GL`, `P&L`, `Production`) and map columns by header text, tolerating normal variation in capitalization, spacing, and punctuation. Full detail in `references/data-extraction.md`.

## Units produced vs. units sold

Keep these separate throughout. COGS and PVM are based on **units sold**, not units produced. Units produced only matters when converting a production-based actual cost into a per-unit figure before restating it on a units-sold basis (Path B in `references/data-extraction.md`). Ending inventory can help explain the gap between the two when they diverge materially.

## Output

The deliverable is the **same workbook, updated in place** (or saved as a new version if the user prefers not to overwrite the original) — not a standalone chat report. It gets new sheets: a Volume Data sheet, one sheet per cost line with the full per-product PVM table (both methods), a Summary Bridge sheet with the nine-step waterfall chart, a Verification sheet, and a Commentary sheet. Full sheet-by-sheet structure, chart approach, and formatting rules are in `references/commentary-and-output.md`.

## Handling missing or zero-volume data

If an expected sheet is missing, identify which calculation depends on it, check whether another sheet covers the same ground, and if not, tell the user exactly what's missing rather than substituting standard costs for actual costs or manufacturing a Price or Mix figure from incomplete data.

If a product has no actual record, include it with actual units and actual COGS set to zero rather than dropping it — its volume and mix effects still belong in the bridge.

Never divide by zero to get a per-unit cost. If budget units are zero (a product introduced during the actual period) or actual units are zero (a product discontinued during the period), flag it as a zero-volume case and let the mix framework capture its contribution instead of forcing a per-unit calculation. Full treatment in `references/data-extraction.md`.

## Final quality standard

A completed analysis should let management answer: how much did COGS differ from budget, how much of that came from Volume, Price, and Mix (under both methods), which cost line and which products drove it, whether each driver was favorable or unfavorable, whether the two decomposition methods and the GL both reconcile, and what to investigate next. The output should explain the drivers, not just reproduce the tables.
