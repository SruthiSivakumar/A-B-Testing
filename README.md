# A-B-Testing
A/B Testing Essentials - Toolkits, sample codes, other learning resources

# Part 1: Simple test kit 

# A/B Test Planner

An Excel workbook for planning an A/B test and checking it afterward. It is built for people who do not work with statistics. You type in a few numbers and read plain-English answers.

It works in Excel, Google Sheets, and LibreOffice. It needs no macros or add-ins.

## What it answers

**Before the test**
- How many customers do I need in Control and in Treatment?
- How many days should I run the test?
- Is my planned test big enough to spot the improvement I care about?

**After the test**
- Were customers split the way I planned? (sample ratio mismatch, or SRM, check)
- Did the Treatment beat the Control, or is the difference just luck?

## How to use it

1. Download `ab_test_simple_toolkit.xlsx` and open it.
2. Go to **Plan Test**. Fill in the yellow cells and read the green results.
3. Run your test.
4. Go to **Check Results**. Fill in the yellow cells with what happened and read the two verdicts.

The **Start Here** tab explains the terms in plain language.

## Inputs

### Plan Test

| Input | Meaning |
|---|---|
| Metric type | Conversion rate (share of customers who buy, click, or sign up) or Average value (such as revenue per customer) |
| Current conversion rate or average value | Where the metric stands before the test |
| Spread of the average value | Standard deviation. Only needed for Average value |
| Smallest improvement worth detecting | Relative change, for example 10% |
| Share of customers in Control and in Treatment | Any two numbers, for example 50 and 50, or 90 and 10 |
| Customers entering the test per day | Both groups together |
| Days you plan to run the test | Use whole weeks |

Confidence (95%), power (80%), and the split-check alert level (0.01) are preset in a settings block. Most people should leave them alone.

### Check Results

| Input | Meaning |
|---|---|
| Customers in Control and in Treatment | Actual group sizes at the end of the test |
| Conversion rate or average value after the test | For Control and for Treatment |

The metric type, planned split, and spread are carried over from Plan Test.

## Outputs

**Plan Test**
- Customers needed in Control, in Treatment, and in total
- Days needed at your daily traffic, and a recommended run time rounded up to whole weeks (minimum 7 days)
- Customers you will get in your planned days
- Chance of spotting the improvement (power) and the smallest improvement you can reliably spot
- A verdict: the plan is big enough, or it is too short

**Check Results**
- Split check: planned share, actual share, p-value, and an OK or PROBLEM message
- Change versus Control, the likely range of the real change, and the chance the difference is luck
- A verdict: Treatment is better, Treatment is worse, no clear winner, or fix the split problem first

If the split check fails, the result verdict tells you not to trust the test.

## Worked examples

These use the default values in the workbook.

| Scenario | Inputs | Result |
|---|---|---|
| Plan a conversion test | 5% current rate, 10% improvement, 50/50 split, 5,000 customers per day | 31,231 customers per group, 13 days needed, 14 days recommended |
| Check a plan | Same test run for 14 days | 84% chance of spotting a 10% improvement. The plan is big enough |
| Split check passes | Planned 50/50, observed 10,000 and 9,900 | p = 0.48, split is OK |
| Split check fails | Planned 50/50, observed 10,000 and 8,000 | p is near zero, do not trust the result |
| Clear win | Control 5.0%, Treatment 5.7%, about 10,000 each | Treatment is better. Likely real change is +1.5% to +26.5% |
| No clear winner | Control 5.0%, Treatment 5.1%, 10,000 each | No clear winner. This test could only spot changes of 17.3% or more |

## How it calculates

The workbook uses the standard normal approximation for comparing two groups, with a two-sided test.

For conversion rates, the Control group size is:

```
n_control = (z_confidence + z_power)^2 * [p1(1-p1) + p2(1-p2)/k] / (p2 - p1)^2
```

For average values:

```
n_control = (z_confidence + z_power)^2 * sigma^2 * (1 + 1/k) / delta^2
```

Here `p1` is the current rate, `p2` is the rate after the improvement, `sigma` is the standard deviation, `delta` is the difference to detect, and `k` is the Treatment size divided by the Control size. This handles uneven splits.

The split check is a chi-square goodness-of-fit test with one degree of freedom. The result check compares the two groups with a z-test.

The workbook uses `NORMSINV`, `NORMSDIST`, and `CHIDIST` so it behaves the same in older Excel, current Excel, and Google Sheets.

## Limits

- Each customer is assumed to be placed in one group at random, on their own. Tests on whole stores, cities, or regions need a different method.
- For very low conversion rates (under 1%) and small tests, treat the sample sizes as a rough guide.
- Average values need a standard deviation. Without a good estimate, the sample size is a guess.
- The sample sizes assume one analysis at the end. Stopping early after repeated looks makes false wins more likely.
- The split check supports two groups. For tests with more than one Treatment, run the check on Control against each Treatment separately.

## Compatibility

Formulas were recalculated in LibreOffice with zero errors, and sample sizes and p-values were checked against an independent calculation in Python. The functions used are available in Excel 2007 and later and in Google Sheets. The workbook has not been opened in Excel itself.

## Contributing

Issues and pull requests are welcome. If you report a calculation problem, include the inputs you used and the output you expected.

## License

MIT.

# Part 2 - More elaborate tool kit.
# A/B Test Power & SRM Toolkit

An Excel workbook for planning A/B tests and checking that a test was run correctly. It calculates sample size, achieved power, and minimum detectable effect (MDE), and it runs a sample ratio mismatch (SRM) check. Every calculation is a live formula, so you can reuse it on any experiment by changing the yellow input cells.

It needs no macros or add-ins and works in Excel and Google Sheets.

## What it does

| Tab | Question it answers |
|---|---|
| **README** | How to use the workbook, definitions, formulas, color legend |
| **SS - Proportions** | How many users per group do I need to detect a change in a conversion-style metric? |
| **SS - Means** | How many users per group do I need to detect a change in a continuous metric? |
| **Power & MDE Calculator** | Given my sample size, what power do I have, and what is the smallest effect I can detect? |
| **SRM Check** | Was traffic split between groups the way I intended? |

## Quick start

1. Download `ab_test_power_srm_toolkit.xlsx`.
2. Open the tab you need.
3. Edit the yellow cells. Every other cell is a formula and updates automatically.
4. Read the green cells for the results.

Each tab ships with example values so you can see it working before you enter your own numbers. To keep a record of a specific test, right-click the tab, choose Move or Copy, and tick Create a copy.

## Tab details

### SS - Proportions

Use this for binary metrics such as conversion rate, click-through rate, sign-up rate, or retention.

- **Inputs:** baseline rate, MDE type (relative lift or absolute percentage points), MDE value, alpha, power, one-sided or two-sided test, number of groups, Bonferroni correction (yes or no), daily traffic per group.
- **Outputs:** expected treatment rate, adjusted alpha, z-scores, required sample size per group, total sample size, estimated test duration in days.

### SS - Means

Use this for continuous metrics such as revenue per user, order value, or time on page.

- **Inputs:** baseline mean, baseline standard deviation, MDE type (relative lift or absolute value), MDE value, alpha, power, test type, number of groups, Bonferroni correction, daily traffic per group.
- **Outputs:** effect size to detect, adjusted alpha, z-scores, required sample size per group, total sample size, estimated test duration.

### Power & MDE Calculator

Four blocks for tests that are already planned or running:

- **A. Achieved power, proportions.** Power at a fixed sample size for a given baseline and treatment rate.
- **B. Achieved power, means.** The same for continuous metrics.
- **C. MDE, proportions.** The smallest absolute and relative effect you can detect at a fixed sample size.
- **D. MDE, means.** The same for continuous metrics.

Achieved power cells turn green at 80% or above and red below 80%.

### SRM Check

Compares observed group counts against the intended split using a chi-square goodness-of-fit test. It supports 2 to 5 groups.

- **Inputs:** number of groups, p-value threshold (default 0.01), and for each group a label, an expected ratio, and an observed count.
- **Outputs:** expected counts, chi-square statistic, degrees of freedom, p-value, and a flag that reads either `No SRM detected` (green) or `SRM DETECTED` (red).

Expected ratios can be any positive numbers. `50` and `50` work the same as `1` and `1`, and `90` and `10` covers an unequal split. Leave unused rows at 0.

If SRM is detected, do not interpret the test results. The groups are not comparable, so any difference in the metric may come from the mismatch itself. Common causes are asymmetric bot filtering, redirect or caching bugs, client-side randomization that fails on some browsers or devices, and logging that drops events differently by group. Fix the cause and rerun the test.

## Methods

The workbook uses the normal approximation (z-test).

**Sample size, proportions**

```
n = (z_alpha + z_beta)^2 * [p1(1 - p1) + p2(1 - p2)] / (p2 - p1)^2
```

**Sample size, means**

```
n = 2 * (z_alpha + z_beta)^2 * sigma^2 / delta^2
```

**Achieved power, proportions**

```
power = Phi( sqrt(n) * |p2 - p1| / sqrt(p1(1 - p1) + p2(1 - p2)) - z_alpha )
```

**Achieved power, means**

```
power = Phi( sqrt(n / 2) * |mu2 - mu1| / sigma - z_alpha )
```

**MDE, proportions (absolute)**

```
MDE = (z_alpha + z_beta) * sqrt( 2 * p1(1 - p1) / n )
```

**MDE, means (absolute)**

```
MDE = (z_alpha + z_beta) * sigma * sqrt(2 / n)
```

**SRM**

```
chi2 = sum( (observed - expected)^2 / expected ),  df = groups - 1
```

For two-sided tests, `z_alpha = NORMSINV(1 - alpha/2)`. For one-sided tests, `z_alpha = NORMSINV(1 - alpha)`. With Bonferroni correction on, alpha is divided by (groups - 1) before the z-score is calculated.

The workbook uses `NORMSINV`, `NORMSDIST`, and `CHIDIST` instead of the newer `NORM.S.INV` family so it behaves the same in older Excel versions, current Excel, and Google Sheets.

## Worked examples

These match the default values in the workbook.

| Scenario | Inputs | Result |
|---|---|---|
| Sample size, proportions | 10% baseline, 10% relative lift, alpha 0.05, power 80%, two-sided | 14,749 per group |
| Sample size, means | Mean 50, SD 15, 10% relative lift, alpha 0.05, power 80%, two-sided | 142 per group |
| Achieved power, proportions | 10% vs 11%, 2,000 per group, alpha 0.05 | 17.7% (underpowered) |
| MDE, proportions | 10% baseline, 2,000 per group, power 80% | 2.7 percentage points (26.6% relative) |
| SRM | 50/50 split, observed 5,000 and 4,850 | chi-square 2.28, p = 0.131, no SRM |
| SRM | 50/50 split, observed 5,000 and 4,000 | chi-square 111.1, p < 0.0001, SRM detected |

The last SRM row is not the default; change the treatment count to 4,000 to reproduce it.

## Limitations

- **Independent observations.** The calculators assume users are randomized individually. They do not handle clustered designs such as geo-level tests, where a method like GeoLift is more appropriate.
- **Normal approximation.** For proportions, aim for at least 10 expected successes and 10 expected failures per group. For very low baseline rates (under 1%) or small samples, treat the sample size as a lower bound and consider an exact test.
- **Equal variance.** The means calculators assume the same standard deviation in both groups.
- **MDE for proportions** uses the baseline variance for both groups. This is a close approximation when the effect is small relative to the baseline.
- **Bonferroni is conservative.** It controls the family-wise error rate but costs power when you have many treatment arms. If that matters, apply a different correction outside this workbook.
- **Fixed horizon.** The sample sizes assume one analysis at the end. Stopping early after repeated looks inflates the false positive rate and needs a sequential method.

## Contributing

Issues and pull requests are welcome. If you report a calculation problem, include the inputs you used and the output you expected.

## License

MIT.
