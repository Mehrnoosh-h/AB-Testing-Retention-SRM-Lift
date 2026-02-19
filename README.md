# Cookie Cats A/B Test — Gate Move (Level 30 → 40)

Beginner-to-solid A/B testing project using real mobile game data. The impact of moving the in-game “gate” from level 30 (control) to level 40 (treatment) is evaluated on 1-day and 7-day retention, with effect sizes, confidence intervals, and practical impact.

Dataset / inspiration notebook: Kaggle — “Introduction to A/B Testing” (Cookie Cats)

---

## 1) Experiment Overview

Goal: Measure the impact of moving the gate from level 30 to level 40 on user retention.

Variants
- Control: gate_30
- Treatment: gate_40

Primary Metrics
- retention_1 — Day-1 retention (came back 1 day after install)
- retention_7 — Day-7 retention (came back 7 days after install)

Secondary / Diagnostic Metric
- sum_gamerounds — number of game rounds played in the first 14 days

---

## 2) Data Dictionary

- userid: unique player identifier  
- version: gate_30 (control) or gate_40 (treatment)  
- sum_gamerounds: total rounds in first 14 days after install  
- retention_1: boolean; returned after 1 day  
- retention_7: boolean; returned after 7 days  

---

## 3) Methods

### 3.1 SRM (Sample Ratio Mismatch) Check
Before testing outcomes, the traffic split is verified against the intended allocation (assumed 50/50). A chi-square goodness-of-fit test is used:

\[
\chi^2=\sum_i \frac{(O_i - E_i)^2}{E_i}
\]

### 3.2 Retention Tests (Two-Proportion Z-Test)
For each retention metric, the following hypotheses are tested:

- H0: p40 = p30
- H1: p40 != p30

Reported outputs:
- retention rates per group
- absolute lift: Δ = p40 − p30
- 95% CI for lift (normal approximation)
- p-value (two-sided)

### 3.3 Multiple Testing Correction (Holm)
Since two metrics (Day-1 and Day-7 retention) are tested, family-wise error is controlled using Holm–Bonferroni.

### 3.4 Bootstrap CI (Robustness Check)
Bootstrap resampling within each group (resample users with replacement) is used to estimate a nonparametric 95% CI for the lift.

### 3.5 Engagement Check (sum_gamerounds)
Because play counts are highly skewed with outliers, comparisons include:
- median + percentiles (p25, p50, p75, p90, p95, p99)
- optional Mann–Whitney U test + common-language effect size P(X40 > X30)

---

## 4) Key Results

### 4.1 Sample Sizes
- gate_30: 44,700
- gate_40: 45,489
- Total: 90,189

### 4.2 SRM Check
- chi-square = 6.90
- SRM p-value = 0.0086 → SRM flag
  - The imbalance is small (~0.87 pp), but statistically detectable; note as a data-quality risk.

### 4.3 Retention (Rates, Lift, CI, Significance)

Day-1 retention
- Control: 44.8188% (20034 / 44700)
- Treatment: 44.2283% (20119 / 45489)
- Lift (treat − control): −0.5905 pp
- 95% CI: [−1.2392, +0.0582] pp
- p-value: 0.0744 (not significant at 0.05)

Day-7 retention
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

### 4.6 Engagement (sum_gamerounds)
- Medians: gate_30 = 17 vs gate_40 = 16 (difference = −1 round)
- Percentiles are nearly identical across groups
- Mann–Whitney p ≈ 0.0502 (borderline), common-language effect P(gate_40 > gate_30) ≈ 0.496 (≈ no difference)
- Extreme outliers exist; median/percentiles are more reliable than mean.

---

## 5) Visualization

<p align="center">
  <img width="1182" height="753" alt="image" src="https://github.com/user-attachments/assets/1a451492-de72-4b1e-a285-92fb25fa3563" />
       alt="image"
       style="width:300px; max-width:70%; height:auto;" />
</p>


The retention lift chart shows the estimated change in retention when moving from gate_30 (control) to gate_40 (treatment). Each bar is the absolute lift (gate_40 − gate_30) for Day-1 and Day-7 retention, and the vertical error bars show the 95% confidence interval. Values below zero indicate lower retention under gate_40. In this experiment, the Day-1 lift is slightly negative but its confidence interval crosses zero (inconclusive), while the Day-7 lift is negative and its confidence interval stays below zero, indicating a statistically reliable decrease in 7-day retention with the gate moved to level 40.

---

## 6) Final Summary Table
  
<p align="center">
  <img width="1400" height="100" alt="image" src="https://github.com/user-attachments/assets/acf44cbb-675b-46fa-89b9-e100869c1797" <p align="center"
   width="300"
       alt="Summary table" />
</p>

---

## 7) Conclusion / Recommendation

- Day-1 retention: small decrease in gate_40, not statistically significant.
- Day-7 retention: statistically significant decrease in gate_40 (also robust via bootstrap and Holm correction).
- Practical significance: the Day-7 decrease corresponds to ~820 fewer retained users per 100k installs.
- Recommendation: do not move the gate to level 40 if retention is the priority.

Caveat: SRM flagged; results should be interpreted with awareness of potential allocation/logging issues.

---

## 8) How to Run

1. Create/activate environment
2. Install dependencies (example):
   ```bash
   pip install pandas numpy scipy statsmodels matplotlib
