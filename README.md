# Problem-Set-3-Econ_3916_PS3
# Problem Set 3: The Number Nobody Can Reproduce

ECON 3916 · Applied Data Analytics · Northeastern University

A dashboard reported average revenue per customer in the West region as \$1,240, and a director quoted it in a board deck. Finance's own query on the same warehouse gave \$812. This notebook rebuilds the warehouse in DuckDB from a known data-generating process, reproduces both numbers in SQL and finds the one difference between the two queries. It then tests whether the West's account-management programme caused the West's higher revenue.

**Notebook:** [`Econ_3916_PS3.ipynb`](Econ_3916_PS3.ipynb)

## The writeup for the director

**To:** the Director · **Re:** \$1,240 or \$812?

**Which number is right.** \$812. "Average revenue per customer" should count each West customer once, and Finance's query does.

**Why the two differed.** The dashboard's query attached every order to its customer before averaging, so a customer with 40 orders was counted 40 times. In our rebuild, 3,000 West customers became 19,618 rows. Our biggest spenders order most, so they dominated the average, pushing it up by half. Nobody's arithmetic was wrong. The query answered a different question.

**Does the programme work?** The regional gap cannot tell us. West was given the larger accounts, and larger accounts spend more with or without an account manager. In a simulation where we know the programme adds \$40 per customer, the raw West-versus-rest gap was \$386. Comparing accounts of similar size recovered \$38. Also controlling for support tickets, which the programme itself generates, flipped it to −\$30. Real data can only be adjusted for what we measured, so the honest answer is "not yet known". A pilot in another region, with accounts assigned to the programme at random, would settle it.

**Before a number goes on a slide:**
1. Every metric names its unit (customer, order), and an automated check confirms one row per unit; a join that adds rows blocks publication.
2. Regional reports show Unknown as its own line, and `!=` on a nullable column fails review.
3. Causal claims control only for things measured before the programme began.

## What it shows

| | Result |
|---|---|
| Finance query (one row per customer) | \$789.20 over 3,000 West customers, inside the \$780–\$845 band around \$812 |
| Dashboard query (customers joined to orders) | \$1,182.40 over 19,618 rows, inside the \$1,160–\$1,320 band around \$1,240 |
| Where the gap comes from | the join repeats each customer once per order; the dashboard number equals Σ(revenue × orders) / Σ orders exactly |
| `WHERE region != 'West'`, with 14.6% of regions NULL | 1,710 rows instead of 2,441: 731 customers dropped without an error |
| Programme effect, built into the simulation | +\$40 per customer |
| Naive West-versus-rest gap | \$386 |
| Adjusted for account size (the confounder) | \$38, 95% CI \$33 to \$43 |
| Also adjusted for support tickets (caused by the programme and by revenue) | −\$30, 95% CI −\$37 to −\$24: the wrong sign |
| AI-written query reviewer | catches the \$1,240 fan-out and the `!=` null filter; misses the same fan-out written inside a CTE |

## Structure

1. **Reproduce both numbers.** Two seeded DuckDB tables, both queries with their row counts, and a fan-out check that fires on the dashboard query and stays silent on the correct ones.
2. **The rows that disappear.** About 15% of regions set to NULL, the three-valued logic that drops them, the query the analyst meant, and a window function that puts each customer next to their region's average without changing the row count.
3. **The causal question.** A simulated programme with a known effect, where account size is a confounder and support tickets are a collider, plus a 500-run check that the result is not luck.
4. **AI-assisted query review.** A reviewer written with Claude Opus 5.5 from a P.R.I.M.E. prompt, scored against the Phase 1 and 2 queries and three probe queries. The unedited transcript is in the notebook.

## Running it

Open the notebook in Google Colab and choose **Runtime › Run all**. It installs `duckdb`; numpy, pandas and statsmodels come preinstalled. Every draw comes from one seeded generator (`numpy.random.default_rng(42)`), so a rerun reproduces every number.
