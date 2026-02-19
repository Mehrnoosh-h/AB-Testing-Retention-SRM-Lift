# AB-Testing-Retention-SRM-Lift

# Cookie Cats A/B Test — Gate Move (Level 30 → 40)

Beginner-to-solid A/B testing project using real mobile game data. We evaluate whether moving the in-game “gate” from level 30 (control) to level 40 (treatment) changes 1-day and 7-day retention, and we quantify the effect with confidence intervals and practical impact.

Dataset / inspiration notebook: Kaggle — “Introduction to A/B Testing” (Cookie Cats)

---

## 1) Experiment Overview

**Goal:** Measure the impact of moving the gate from level 30 to level 40 on user retention.

**Variants**
- **Control:** `gate_30`
- **Treatment:** `gate_40`

**Primary Metrics**
- `retention_1` — Day-1 retention (came back 1 day after install)
- `retention_7` — Day-7 retention (came back 7 days after install)

**Secondary / Diagnostic Metric**
- `sum_gamerounds` — number of game rounds played in the first 14 days

---

## 2) Data Dictionary

- `userid`: unique player identifier  
- `version`: `gate_30` (control) or `gate_40` (treatment)  
- `sum_gamerounds`: total rounds in first 14 days after install  
- `retention_1`: boolean; returned after 1 day  
- `retention_7`: boolean; returned after 7 days  

---

## 3) Methods

### 3.1 SRM (Sample Ratio Mismatch) Check
Before testing outcomes, we verify the traffic split matches the intended allocation (assumed 50/50).  
We use a chi-square goodness-of-fit test:

\[
\chi^2=\sum_i \frac{(O_i - E_i)^2}{E_i}
\]

### 3.2 Retention Tests (Two-Proportion Z-Test)
For each retention metric, we test:

- **H0:** \( p_{40} = p_{30} \)
- **H1:** \( p_{40} \neq p_{30} \)

We report:
- retention rates per group
- absolute lift: \( \Delta = p_{40} - p_{30} \)
- 95% CI for lift (normal approximation)
- p-value (two-sided)

### 3.3 Multiple Testing Correction (Holm)
Since we test two metrics (Day-1 and Day-7 retention), we control family-wise error using Holm–Bonferroni.

### 3.4 Bootstrap CI (Robustness Check)
We bootstrap within each group (resample users with replacement) to estimate a nonparametric 95% CI for the lift.

### 3.5 Engagement Check (`sum_gamerounds`)
Because play counts are highly skewed with outliers, we compare:
- median + percentiles (p25, p50, p75, p90, p95, p99)
- optional Mann–Whitney U test + common-language effect size \(P(X_{40} > X_{30})\)

---

## 4) Key Results

### 4.1 Sample Sizes
- `gate_30`: 44,700
- `gate_40`: 45,489
- Total: 90,189

### 4.2 SRM Check
- χ² = 6.90
- SRM p-value = 0.0086 → SRM flag
  - The imbalance is small (~0.87 pp), but statistically detectable; note as a data-quality risk.

### 4.3 Retention (Rates, Lift, CI, Significance)

**Day-1 retention**
- Control: 44.8188% (20034 / 44700)
- Treatment: 44.2283% (20119 / 45489)
- Lift (treat − control): −0.5905 pp
- 95% CI: [−1.2392, +0.0582] pp
- p-value: 0.0744 (not significant at 0.05)

**Day-7 retention**
- Control: 19.0201% (8502 / 44700)
- Treatment: 18.2000% (8279 / 45489)
- Lift (treat − control): −0.8201 pp
- 95% CI: [−1.3282, −0.3121] pp
- p-value: 0.00155
- Holm-adjusted p-value: 0.00311 (still significant)

### 4.4 Practical Impact (Per 100,000 installs)
- Day-1: ~590 fewer retained users per 100k installs
- Day-7: ~820 fewer retained users per 100k installs

### 4.5 Bootstrap CI for Lift (Robustness)
- Day-1 bootstrap 95% CI: [−1.2337, +0.0548] pp (includes 0)
- Day-7 bootstrap 95% CI: [−1.3378, −0.3141] pp (entirely < 0)

### 4.6 Engagement (`sum_gamerounds`)
- Medians: gate_30 17 vs gate_40 16 (difference −1 round)
- Percentiles are nearly identical across groups
- Mann–Whitney p ≈ 0.0502 (borderline), but common-language effect:
  - \(P(\text{gate\_40} > \text{gate\_30}) \approx 0.496\) (≈ no difference)
- Note: extreme outliers exist; median/percentiles are more reliable than mean.

---

## 5) Conclusion / Recommendation

- **Day-1 retention:** small decrease in gate_40, not statistically significant.
- **Day-7 retention:** statistically significant decrease in gate_40 (also robust via bootstrap and Holm correction).
- **Practical significance:** the Day-7 decrease corresponds to ~ 820 fewer retained users per 100k installs.
- **Recommendation:** Do not move the gate to level 40 if retention is the priority.

**Caveat:** SRM flagged; results should be interpreted with awareness of potential allocation/logging issues.

---

## 6) How to Run

1. Create/activate your environment
2. Install dependencies (example):
   ```bash
   pip install pandas numpy scipy statsmodels matplotlib
