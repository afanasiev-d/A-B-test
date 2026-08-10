# A/B Test — Mobile Payment Screen Redesign

**Context.** A consumer product monetised through in-app purchases. The design team proposed
a new mobile payment screen intended to increase completed payments. It was rolled out to a
subset of users from **24 July, 00:00**, with the remainder kept on the existing screen.

**Question.** Ship the change to everyone, or reject it?

**Answer.** Reject. Paying share was **significantly lower** in the treatment arm
(χ² = 8.81, p = 0.0122). An A/A test run beforehand confirmed the assignment mechanism was
unbiased, so the effect is attributable to the redesign.

---

## 1. Target metric

The business outcome is the number of successful payments — a discrete variable following a
binomial distribution $B(n, p)$. Rather than test raw counts, the analysis uses the
normalised proxy **paying share**: the fraction of users in an arm who completed a
*successful* payment.

For the binomial proportion:

$$\mathbb{E}[\hat{p}] = \frac{np}{n} = p,
\qquad
\operatorname{Var}(\hat{p}) = \frac{np(1-p)}{n^{2}} = \frac{p(1-p)}{n}$$

Normalising by arm size makes the two groups comparable when they differ in size, and the
variance expression above is what the power calculation in §3 is built on.

---

## 2. Constructing a valid control group

This is the step that determines whether the whole test is meaningful.

The dataset marks treatment users with `split_group = 1`, but `0` marks **everyone else** —
including users whose payment date falls *before* 24 July. Comparing post-launch treatment
users against a control pool containing pre-launch behaviour would confound the treatment
effect with any change over time.

The control pool was therefore restricted to non-treatment users who either:

- have a payment date on or after 24 July, **or**
- registered on or after 24 July (covering users who never attempted a payment but sit in
  the correct time window).

One treatment-arm record was excluded as anomalous: it shows a payment on 23 July, before
the experiment began.

**Why this matters.** Paying share shows a rising trend across the observation window. That
trend alone says nothing about the redesign — it could equally reflect seasonality, wage
cycles, macroeconomic conditions, or other concurrent product changes. Aligning both arms to
the same time window is what removes that alternative explanation.

---

## 3. Power analysis — required sample size

Sample size was fixed *before* testing, using the standard two-proportion formula:

$$n = \frac{\left[
z_{1-\alpha/2}\sqrt{\bar{p}(1-\bar{p})\left(1+\tfrac{1}{k}\right)} +
z_{1-\beta}\sqrt{p_1(1-p_1) + \tfrac{p_2(1-p_2)}{k}}
\right]^{2}}{\mathrm{MDE}^{2}}$$

| Parameter | Value |
|---|---|
| $\bar{p}$ — pooled success probability | 0.167 |
| $p_1$ — control | 0.230 |
| $p_2$ — treatment | 0.103 |
| MDE $= \lvert p_1 - p_2 \rvert$ | 0.127 |
| $z_{1-\alpha/2}$ — significance 95% | 1.96 |
| $z_{1-\beta}$ — power 80% | 0.84 |
| $k$ — arm size ratio | 1 (equal arms) |
| **Required n per arm** | **≈ 134** |

The requirement is small because the effect being detected is large — MDE sits in the
denominator, and a 12.7 pp gap between arm conversion rates is a substantial difference.

Deliberately sampling down to the minimum sufficient size does not degrade test quality:
significance and power were both fixed in advance at conventional levels. Large-sample tests
are expensive, and running smaller adequately-powered tests means **more experiments per unit
of time** — which is what actually finds the change that moves the metric.

---

## 4. Stratified sampling

Exploratory analysis showed material differences in the key metrics across `age_group`, and
**one-way ANOVA confirmed the between-group differences are statistically significant**. Age
is therefore a confounder for payment propensity.

Sampling was stratified on `age_group` within the post-24-July population. The justification
is variance decomposition — total variance under simple random sampling splits into
within-stratum and between-stratum components:

$$\operatorname{Var}_{\text{SRS}} =
\underbrace{\operatorname{Var}_{\text{within}}}_{\text{retained}} +
\underbrace{\operatorname{Var}_{\text{between}}}_{\text{removed by stratification}}$$

Removing the between-stratum term shrinks the sampling variance, which lowers the
probability of Type I and Type II error at a **fixed** sample size — equivalently, it lets
the same effect be detected with fewer users.

---

## 5. Choice of test

A **Pearson χ² test of independence** on the contingency table of arm × payment outcome:

$$E_{ij} = \frac{R_i C_j}{N},
\qquad
\chi^{2} = \sum_{i,j}\frac{(O_{ij}-E_{ij})^{2}}{E_{ij}},
\qquad
\mathrm{df} = (r-1)(c-1) = 2$$

> **Table layout:** 2 arms × 3 payment-outcome categories, hence 2 degrees of freedom.

Chosen over **Fisher's exact test** because the sample is not small enough to make
enumerating all permutations practical, and because the target metric is a binomial
proportion for which the χ² approximation is well behaved at these expected cell counts.

---

## 6. Results

### 6.1 A/A control test

Before the live comparison, the control population was split into two random subgroups and
tested against each other. Under correct randomisation there should be no detectable
difference.

| Statistic | Value |
|---|---|
| χ² | 0.4177 |
| p-value | **0.8115** |

p > 0.05 → cannot reject H₀. The assignment is unbiased and the procedure has no structural
false-positive tendency, so a significant A/B result can be read as a genuine effect.

### 6.2 A/B test

| Statistic | Value |
|---|---|
| Paying share — control (A) | 23.0% |
| Paying share — treatment (B) | 10.3% |
| Absolute difference | −12.7 pp |
| χ² | 8.8101 |
| p-value | **0.0122** |

p < 0.05 → reject H₀. Control paying share is **higher** than treatment, and the difference
is statistically significant.

---

## 7. Decision

**Reject the redesign.** The new mobile payment screen significantly *reduced* the share of
users completing a successful payment — the opposite of its intended effect.

If the redesign is strategically necessary, re-test a modified version with the payment
funnel instrumented step by step, to locate where users drop out.

### Limitations

- Single metric over a single time window; no assessment of novelty effects or long-run
  behaviour.
- Segment-level effects are described but not formally tested; repeated subgroup testing
  would require multiple-comparison correction.
- Revenue per paying user is not analysed — a change can reduce conversion while raising
  average basket size.

---

## Reproducing

```bash
pip install pandas numpy scipy statsmodels matplotlib seaborn
jupyter notebook ab_test.ipynb
```
