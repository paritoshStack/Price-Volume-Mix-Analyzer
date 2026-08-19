# PVM calculations

Run everything in this file independently for each of the three cost lines: Direct Materials, Direct Labor, Variable Overhead. Calculate **both** methods below for every line — the two-step and the three-way — and carry both into the final output.

## Step 1: volume comparison (shared across all three cost lines)

Build a table comparing budget vs. actual units sold, by product. This table is the same regardless of which cost line you're analyzing next, since the units sold don't change based on which cost you're looking at.

Δ Volume = Actual Units Sold − Budget Units Sold

## Step 2: per-unit costs

Budget $/unit = Budget Total Cost ÷ Budget Units

Actual $/unit, when actual COGS is already on a units-sold basis (Path A, or Path B after conversion):

Actual $/unit = Actual COGS ÷ Units Sold

## Method 1: two-step decomposition (Volume + Price)

This is the simpler, older decomposition. It isolates volume and price effects but folds any product-mix effect into the volume number.

**Volume Variance**, per product:

Volume Variance = (Actual Units − Budget Units) × Budget $/unit

This isolates the impact of selling more or fewer units of that product, at that product's own budgeted cost per unit.

**Price Variance**, per product:

Price Variance = Actual Units × (Actual $/unit − Budget $/unit)

This isolates the impact of the actual cost per unit differing from budget, applied to the actual volume of that product.

**Total (two-step):**

Total Variance = Volume Variance + Price Variance

Sum both variances across every product to get the two-step total for the cost line.

## Method 2: three-way decomposition (Volume + Mix + Price)

This separates the effect of total volume changing from the effect of the product mix shifting — something the two-step method can't distinguish, since it prices each product's volume change at that product's own rate.

Conceptual bridge: Budget → Volume → Mix → Price → Actual.

**Budget COGS:**

Budget COGS = Σ (Budget Units_i × Budget Unit Cost_i)

**Volume step.** Holds the budget mix and budget unit costs constant, and isolates the effect of total volume moving from budget to actual:

Budget Average Cost per Unit = Budget COGS ÷ Budget Total Units

Volume Variance = (Actual Total Units − Budget Total Units) × Budget Average Cost per Unit

**Mix step.** Isolates the effect of the product composition changing at actual total volume, holding unit costs at budget levels:

Budget Mix Share_i = Budget Units_i ÷ Budget Total Units

Expected Units at Budget Mix_i = Actual Total Units × Budget Mix Share_i

Mix Variance = Σ [(Actual Units_i − Expected Units at Budget Mix_i) × Budget Unit Cost_i]

**Price step.** Same concept as the two-step Price variance, and always comes out numerically identical to it (see the identity below):

Price Variance = Σ [Actual Units_i × (Actual Unit Cost_i − Budget Unit Cost_i)]

**Reconciliation:**

Actual COGS − Budget COGS = Volume Variance + Mix Variance + Price Variance

## The cross-method identity — use this as an extra reconciliation check

Because both methods start from the same underlying data, they're mathematically linked, not just two unrelated ways of slicing the same total. Specifically:

**Two-step Volume Variance (summed across products) = Three-way Volume Variance + Three-way Mix Variance**

**Two-step Price Variance (summed across products) = Three-way Price Variance**

In plain terms: the two-step method's "Volume" figure is really a blend of pure volume change and mix shift, priced at each product's own rate. The three-way method takes that same blended figure and splits it cleanly into a pure volume effect (priced at the average rate) and a mix effect (the composition shift, priced at the difference products' own rates imply). Price is identical either way, because neither method disputes what actually happened to unit cost.

Check this identity every time you run both methods. If it doesn't hold, something in the calculation is inconsistent — recheck the budget mix shares and the average-cost calculation before looking anywhere else.

## Reconciliation tolerance

Both decompositions should sum exactly to Actual COGS − Budget COGS, and the cross-method identity above should hold exactly, in principle. In practice, floating-point arithmetic and Excel rounding can introduce small noise.

Treat a reconciliation as passing if it's within **$1 or 0.01% of the relevant total, whichever is larger**. Anything outside that tolerance should be investigated, not silently accepted or absorbed into an unlabeled residual. Common causes: different unit bases between sheets, incorrect budget mix weights, incorrect actual cost allocation, inventory effects, missing or duplicate products, period mismatches, standard prices used in place of true actuals, or unallocated GL costs.

Never force a bridge to reconcile by inserting an unexplained plug — if a residual remains after checking the causes above, label it explicitly as an unreconciled residual and say so in the commentary.

## Sign convention

**Cost lines (DM, DL, VOH):** positive variance = unfavorable (actual costs exceeded budget, which reduces profit). Negative variance = favorable (actual costs came in below budget).

**Volume variance:** negative when fewer units are sold. Fewer units sold means less cost, which is favorable from a pure cost standpoint — but fewer units sold also means less revenue, so COGS volume variance should normally be read alongside the matching revenue variance, not in isolation.

**Price variance:** positive when Actual cost/unit > Budget cost/unit (unfavorable). Negative when Actual cost/unit < Budget cost/unit (favorable).

**Mix variance:** positive when the mix shifts toward higher-cost products (unfavorable for COGS on its own). The same shift can be favorable for revenue or margin if the higher-cost products also carry higher prices or better margins — don't assume a positive COGS mix variance is bad news for the business overall without checking the revenue side.

Always label each variance favorable or unfavorable explicitly in commentary. Don't make the reader infer it from the sign.

## Worked example, fully reconciled

Direct Materials line, two products: A (Basic) and B (Premium).

**Inputs:**

| | Budget units | Budget $/unit | Actual units | Actual $/unit |
|---|---|---|---|---|
| A | 600 | $10.00 | 500 | $10.50 |
| B | 400 | $20.00 | 550 | $19.00 |

Budget COGS: A = 600 × $10.00 = $6,000. B = 400 × $20.00 = $8,000. Total = $14,000.

Actual COGS: A = 500 × $10.50 = $5,250. B = 550 × $19.00 = $10,450. Total = $15,700.

Total Variance = $15,700 − $14,000 = **$1,700 unfavorable**.

**Two-step, per product:**

A — Volume = (500 − 600) × $10.00 = **−$1,000**. Price = 500 × ($10.50 − $10.00) = **$250**.
B — Volume = (550 − 400) × $20.00 = **$3,000**. Price = 550 × ($19.00 − $20.00) = **−$550**.

Check: A total = −1,000 + 250 = −$750, matches $5,250 − $6,000. B total = 3,000 − 550 = $2,450, matches $10,450 − $8,000. ✓

Two-step totals: Volume = −1,000 + 3,000 = **$2,000**. Price = 250 − 550 = **−$300**. Sum = $1,700. ✓ matches total variance.

**Three-way:**

Budget Average Cost per Unit = $14,000 ÷ 1,000 = $14.00.

Volume Variance = (1,050 − 1,000) × $14.00 = **$700**.

Budget Mix Share: A = 600 ÷ 1,000 = 0.60. B = 400 ÷ 1,000 = 0.40.

Expected Units at Budget Mix: A = 1,050 × 0.60 = 630. B = 1,050 × 0.40 = 420.

Mix Variance = (500 − 630) × $10.00 + (550 − 420) × $20.00 = −$1,300 + $2,600 = **$1,300**.

Price Variance = 500 × ($10.50 − $10.00) + 550 × ($19.00 − $20.00) = $250 − $550 = **−$300**.

Three-way total = 700 + 1,300 − 300 = **$1,700**. ✓ matches total variance and the two-step total.

**Cross-method identity check:**

Two-step Volume ($2,000) = Three-way Volume ($700) + Three-way Mix ($1,300) = $2,000. ✓

Two-step Price (−$300) = Three-way Price (−$300). ✓

**Reading the result:** the two-step method says "$2,000 unfavorable from volume, $300 favorable from price." That's true but incomplete — it hides the fact that most of that $2,000 is really a mix shift toward the more expensive Premium product (B), not a genuine increase in total volume. The three-way method shows the real story: only $700 of the $2,000 came from selling more units overall (1,050 vs. 1,000 budgeted); the other $1,300 came from those extra units skewing toward the pricier product. That distinction is exactly why both methods belong in the output side by side — the two-step gives a quick top-line read, the three-way gives the actionable driver.
