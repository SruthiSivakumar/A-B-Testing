# A-B-Testing
A/B Testing Essentials - Toolkits, sample codes, other learning resources
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
