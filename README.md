# A/B Test — Impact of a Product Change on Payment Conversion

**Question:** did the treatment change the share of registered users who go on to make a
successful payment?

**Answer:** yes — and negatively. Paying share fell from **23.09%** (control) to **10.32%**
(treatment), a drop of **12.8 percentage points** (95% CI: 8.9–16.6 pp), a **55% relative
decline**. An A/A test run first confirmed the randomisation was sound, so the effect is
attributable to the treatment rather than to sampling bias.

**Recommendation:** do not roll out.

---

## 1. Data

`raw_data.csv` — 58,938 registered users, 13 fields.

| Field | Description |
|---|---|
| `id_user` | Unique user identifier |
| `date_reg`, `date_payment` | Registration and payment timestamps |
| `split_group` | Experiment arm (0 = control, 1 = treatment) |
| `successful_payment` | Binary outcome flag |
| `amount`, `method` | Payment value and instrument (card, Apple Pay, …) |
| `platform`, `system` | Mobile / desktop / other; OS |
| `gender`, `age_group`, `country_group`, `id_traffic_source` | User covariates |

Registrations concentrate in July (34,255), June (12,512) and August (12,171).

---

## 2. Metric definition

The primary metric is **paying share** — the proportion of users in an arm who complete at
least one successful payment. For user $i$, let $y_i$ be the payment indicator:

$$y_i = \mathbf{1}\left[\text{user } i \text{ made a successful payment}\right]
\in \{0, 1\}$$

The arm-level estimate is then the sample mean of that indicator:

$$\hat{p} = \frac{1}{n}\sum_{i=1}^{n} y_i,
\qquad
\operatorname{Var}(\hat{p}) = \frac{\hat{p}(1-\hat{p})}{n}$$

In the data this corresponds to the `successful_payment` flag aggregated to user level.

Paying share was chosen over average revenue per user because it is bounded, robust to the
heavy right tail in `amount`, and directly reflects the conversion behaviour the treatment
was intended to influence. Revenue is tracked as a secondary metric.

---

## 3. Design

### 3.1 Stratified sampling

Users were sampled from each arm with the **age distribution held fixed**, so that any
observed difference cannot be explained by age composition — a known confounder for payment
propensity.

| Arm | n |
|---|---|
| A — control | 645 |
| B — treatment | 826 |

### 3.2 A/A validation (run *before* the A/B analysis)

The control arm was split into two random subgroups, A₀ and A₁, and tested against each
other. Under a correct randomisation there should be no detectable difference.

> χ² test on A₀ vs A₁ → **p = 0.8115**

No significant difference. The assignment mechanism is unbiased and the test has no
structural false-positive tendency, so a significant A/B result can be read as a real effect.

---

## 4. Statistical test

Paying share is compared across arms with a **Pearson χ² test of independence** on the
2×2 contingency table (arm × paid), with expected counts

$$E_{ij} = \frac{R_i C_j}{N}, \qquad
\chi^2 = \sum_{i,j} \frac{(O_{ij} - E_{ij})^2}{E_{ij}}, \qquad \mathrm{df} = 1$$

Equivalently, as a two-proportion z-test (the two are related by χ² = z²):

$$z = \frac{\hat{p}_A - \hat{p}_B}
{\sqrt{\hat{p}(1-\hat{p})\left(\tfrac{1}{n_A}+\tfrac{1}{n_B}\right)}},
\qquad \hat{p} = \frac{x_A + x_B}{n_A + n_B}$$

Both are two-sided at α = 0.05. The χ² test is appropriate here because all expected cell
counts exceed 5; Fisher's exact test would be required otherwise.

### Results

| Quantity | Value |
|---|---|
| Paying share, control (A) | 23.09% (149 / 645) |
| Paying share, treatment (B) | 10.32% (85 / 826) |
| Absolute difference | **−12.77 pp** |
| Relative change | **−55.3%** |
| 95% CI on the difference | [8.91, 16.63] pp |
| Pooled proportion | 0.1592 |
| z-statistic | 6.64 |
| χ² (1 df) | <!-- RE-RUN AND INSERT --> |
| p-value | <!-- RE-RUN AND INSERT --> |
| Cohen's h | 0.348 (small-to-moderate) |

Confidence interval on the difference of proportions:

$$(\hat{p}_A - \hat{p}_B) \pm z_{1-\alpha/2}
\sqrt{\frac{\hat{p}_A(1-\hat{p}_A)}{n_A} + \frac{\hat{p}_B(1-\hat{p}_B)}{n_B}}$$

The interval excludes zero across its whole range, so the direction of the effect is not in
doubt even at the conservative end.

---

## 5. Conclusion

1. The treatment **reduced** payment conversion by roughly 13 percentage points — over half
   the baseline rate.
2. The A/A test rules out selection bias as an explanation.
3. The effect is large enough to matter commercially, not merely statistically detectable.

**Recommendation: do not ship.** If the feature is strategically necessary, re-test a
modified version and instrument the payment funnel to locate the step at which users drop.

### Limitations

- Single metric, single time window; no long-run or novelty-effect analysis.
- No pre-registered minimum detectable effect, so the design's power was not fixed in advance.
- Segment-level effects (platform, country, traffic source) are described but not formally
  tested — repeated subgroup testing would need multiple-comparison correction.

---

## 6. Reproducing

```bash
pip install pandas numpy scipy matplotlib seaborn
jupyter notebook ab_test.ipynb
```ment group negatively impacted payment behavior.
