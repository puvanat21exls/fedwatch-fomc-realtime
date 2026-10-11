# 🏛️ FedWatch: Real-Time Forecasting of FOMC Rate Decisions

A pre-registered research study of **261 scheduled FOMC meetings (Feb 1994 – Sep 2026)** asking one question: does macroeconomic data, measured **exactly as it was published at the time** (ALFRED real-time vintages), add information about the Fed's decision beyond what a Treasury yield-curve proxy already says? One test was fixed and hash-locked before any result existed. Everything else is labelled exploratory.

> Research study only: it measures information content. It does not trade and is not investment advice.

---

## 📌 Research Question & Core Empirical Findings

Tools such as CME FedWatch read the Fed's next move straight from fed funds futures, and public macro data cannot beat them. The useful question is narrower: **eleven days before a meeting, on the last day before the Fed's blackout, does real-time macro data add anything to market pricing, and where is the market wrong?**

### Key Empirical Results (208 out-of-sample meetings, Oct 2000 – Sep 2026)

* **Macro adds no detectable information about the decision (pre-registered test, $p = 0.30$):** Outside the zero-rate years (n = 136), a logit using the policy state plus three yield-curve gaps scores a log loss of **0.512**. Adding four real-time macro variables gives **0.699**. The difference is not statistically distinguishable from zero.
* **The result does not depend on the forecast day (exploratory):** Re-running the same comparison 7, 4 and 1 day(s) before the meeting, after the final CPI and jobs releases, gives the same verdict ($p$ = 0.25 to 0.34).
* **Five meetings in 2002 explain the gap (descriptive):** The unemployment signal read 1.3–1.6 against a training-sample maximum of 0.53, and the macro model confidently predicted rate moves that never came. Those five meetings are 121% of the total loss gap; across the other 131 the macro model is slightly *better* (−0.04). Trimming extreme inputs narrows the gap to +0.07 but does not close it.
* **Markets' surprises are partly predictable (Clark–West $p = 0.008$):** Real-time data known 11 days ahead predicts the Bauer–Swanson monetary policy surprise out of sample. This holds when Covid meetings are excluded, when inputs are winsorized, and with both. The effect is small (a few basis points; out-of-sample $R^2 \approx -0.03$) and concentrated in time: 77% of the evidence was built up before 2009.

**In one sentence:** the market cannot be out-predicted on what the Fed will do, but when the market is wrong, it is wrong in a slightly predictable way.

![Thirty years of Fed decisions](results/figures/fig1_fed_decisions.png)

---

## 🔬 Methodology & Formula Definitions

### 1. Timing and Label

Each row is one scheduled meeting. Features are measured as of the close of

$$D = \text{meeting start} - 11 \text{ days}$$

the last day before the blackout period. The label is the change in the target rate (upper bound) from the day before the meeting to 3 days after the decision: **Cut** ($\le -10$ bp), **Hold**, or **Hike** ($\ge +10$ bp). Unscheduled moves feed the state features but are not predicted.

### 2. Real-Time Data (ALFRED)

For every revised series, all historical vintages are downloaded. The series "as known on $D$" is every observation whose real-time window contains $D$:

`realtime_start` ≤ D ≤ `realtime_end`

This gives both the publication lag (a release that had not happened yet is absent) and the pre-revision values. Growth rates are computed within one vintage, so index rebasings cancel. Meetings that fall before a series' first archived vintage use that first vintage with release-date rules and are **flagged**: core PCE 20% of meetings, core CPI 9%, CBO $u^{\ast}$ 52%, jobless claims 47%; unemployment, payrolls and GDP 0%.

### 3. Features

**Macro block (4 variables, the treatment):**

$$\text{Taylor gap} = i_{\text{mid}} - \Big[ r^{\ast} + \pi + 0.5(\pi - \pi^{\ast}) + 0.5 \times \big(-2(u - u^{\ast})\big) \Big], \qquad r^{\ast} = \pi^{\ast} = 2$$

$$\text{Inflation momentum} = \left[\left(\frac{P_t}{P_{t-3}}\right)^{4} - 1\right] \times 100 - 2 \qquad \text{(core PCE)}$$

$$\text{Sahm signal} = \bar{u}^{3m}_t - \min_{k = 1..12} \bar{u}^{3m}_{t-k}$$

$$\text{Claims change} = \left(\frac{\overline{\text{ICSA}}_{\text{last 4 wks}}}{\overline{\text{ICSA}}_{\text{4 wks, 13 wks earlier}}} - 1\right) \times 100$$

* **$\pi$**: core PCE inflation, % over 12 months; **$u$**: unemployment rate; **$u^{\ast}$**: CBO noncyclical rate
* **$i_{\text{mid}}$**: midpoint of the federal funds target

**Market block (3):** the 3-month bill, 6-month bill and 2-year Treasury yields, each minus the effective funds rate (5-day median through $D-1$).
**State block (4):** rate change since the previous meeting, change over the last three meetings, days since the last move (capped at 730), and a zero-lower-bound flag.

> **Why only four macro variables?** The test sample has 63 rate moves and 10–15 genuine turning points. A larger feature set (15 macro variables) scores significantly *worse* out of sample ($p = 0.003$).

### 4. Models, Validation and the Locked Test

* **Control:** L2 logistic regression ($C = 0.3$) on state + market. **Treatment:** the same plus the macro block.
* **Walk-forward:** expanding window, first training set of at least 48 meetings, refit every 8 meetings (26 blocks). No meeting is predicted by a model that saw its outcome.
* **Score:** multiclass log loss, $-\ln p(\text{actual outcome})$, probabilities floored at 0.001.
* **Test:** Diebold–Mariano on per-meeting loss differences with Newey–West errors (4 lags), two-sided, $\alpha = 0.10$, on out-of-sample meetings with the target above 0.25%.

The full specification is in [`results/PREREGISTRATION_V5.md`](results/PREREGISTRATION_V5.md); its SHA-256 lock (`c8b679059019…`) is in [`results/PREREG_LOCK.json`](results/PREREG_LOCK.json).

---

## 📊 Results

### Pre-registered Primary Test

| | Log loss | |
|---|---|---|
| Control: state + yield curve | **0.5118** | |
| Treatment: + real-time macro | **0.6989** | |
| Mean difference (treatment − control) | **+0.1871** | $t = +1.05$, $p = 0.295$ |
| Block-bootstrap 90% interval | [−0.044, +0.541] | includes zero |
| **Verdict** | **No detectable difference** | n = 136, 63 moves |

Including the zero-rate years: difference +0.182, $p = 0.140$ (n = 208). Using today's revised data instead of real-time data: $p = 0.237$, the same verdict.

### Horizon Analysis (exploratory)

| Days before meeting | Control | Treatment | Difference | $p$ | Verdict |
|---|---|---|---|---|---|
| **11** (pre-registered) | 0.512 | 0.699 | +0.187 | 0.295 | No detectable difference |
| 7 | 0.544 | 0.729 | +0.185 | 0.334 | No detectable difference |
| 4 | 0.515 | 0.693 | +0.178 | 0.336 | No detectable difference |
| 1 | 0.534 | 0.746 | +0.212 | 0.248 | No detectable difference |

Four days before the meeting, 27% of meetings have a newer CPI release and 100% have newer jobless claims than at 11 days; the verdict does not change. See [`results/HORIZON_EXPLAINED.md`](results/HORIZON_EXPLAINED.md).

### Why Does the Macro Model Score Worse?

![Why did 2002 break the macro model?](results/figures/fig2_why_2002.png)

| | Sum of loss gap | Share | Mean gap of the remaining meetings |
|---|---|---|---|
| All 136 meetings | +25.44 | 100% | +0.187 |
| Top 5 (all 2002) | +30.75 | 121% | −0.041 |
| Top 10 | +33.43 | 131% | −0.063 |

Winsorizing every feature at the 1st/99th percentile of each training block (control and treatment alike) gives 0.582 vs 0.511, a difference of +0.071 ($p = 0.308$). Under the decision rule written before running it, "fat tails drove the null" is **not supported**.

### Turning Points (secondary)

Of 19 out-of-sample turning points (7 reversals, 4 restarts, 8 pauses), the treatment gave the actual outcome a higher probability at 10, the control at 8, with 1 tie (sign test $p = 0.815$). The table is in [`results/turning_points_v5.csv`](results/turning_points_v5.csv).

### Surprise Predictability (secondary, with exploratory robustness)

Target: the raw Bauer–Swanson monetary policy surprise. Predictors known at the anchor: payroll growth, 3-month equity and oil returns, the change in the 2s10s slope, inflation momentum, the Sahm signal and the change in claims. Expanding-window OLS against a zero forecast, Clark–West one-sided test.

| Anchor | Variant | n | OOS $R^2$ | Clark–West $t$ | $p$ |
|---|---|---|---|---|---|
| 1 day before | Baseline | 199 | −17.18 | 2.08 | 0.019 |
| 1 day before | Covid window excluded | 185 | −0.016 | 2.32 | 0.010 |
| 1 day before | Winsorized | 199 | −0.006 | 2.52 | 0.006 |
| 1 day before | Both | 185 | −0.010 | 2.29 | 0.011 |
| **11 days before** | Baseline | 199 | −19.87 | 2.42 | **0.008** |
| **11 days before** | Covid window excluded | 185 | −0.040 | 2.71 | 0.003 |
| **11 days before** | Winsorized | 199 | −0.029 | 2.81 | 0.002 |
| **11 days before** | Both | 185 | −0.022 | 2.69 | 0.004 |

The baseline $R^2$ of about −17 comes from one meeting (Apr 2020), where jobless claims were about 395 standard deviations from their training mean. Under the rule written before the robustness runs, both anchors are **robustly predictable** ($p < 0.10$ in all three variants).

![Economic data could partly predict the market's surprise](results/figures/fig4_surprise_predictability.png)

**Reading it:** the Clark–West test detects a reliable signal, but the forecasts do not beat "no surprise" on average error, and the evidence is uneven: 77% of it predates 2009 and one meeting (Dec 2008) is 27% of the total.

---

## 🧪 Validation & Integrity Checks

* **Pre-registration lock:** the test specification was committed and hashed before the first real-data run; every later run reports whether it still matches the lock.
* **Look-ahead test on real vintages:** every value published *after* $D$ is scrambled and no feature changes (124 fully real-time meetings; 137 flagged-fallback meetings checked on market and state features). A *teeth* check scrambles values known *at* $D$ and confirms the macro features do change (124 of 124), so the test itself is not vacuous.
* **Labels:** an independent recomputation from the raw target-rate files agrees for every meeting (0 disagreements).
* **Re-scoring:** log losses recomputed from the saved predictions reproduce the primary numbers exactly.
* **Calendar:** all 1993–2020 meeting dates were checked against the Federal Reserve's historical pages (4 dates corrected before the lock); 2021–2027 match the Fed's current calendar.
* **Real-time vs revised:** every meeting's macro inputs differ between first-release and revised data (core PCE inflation by 0.22 points on average, up to 1.3).

![The numbers the Fed saw were not the numbers we see today](results/figures/fig3_realtime_vs_revised.png)

---

## 🔮 Live Forecast Record

Forecasts for upcoming meetings are appended to a hash-chained ledger ([`results/forecast_ledger.jsonl`](results/forecast_ledger.jsonl)); editing or removing a past entry breaks the chain. Official forecasts use only information public by the close of the anchor day.

| Meeting | Anchor day | Status |
|---|---|---|
| Oct 27–28, 2026 | Fri Oct 16, 2026 | Scheduled |
| Dec 8–9, 2026 | Fri Nov 27, 2026 | Scheduled |
| Jan 26–27, 2027 | Fri Jan 15, 2027 | Scheduled |
| Mar 16–17, 2027 | Fri Mar 5, 2027 | Scheduled |

The ledger currently holds three unofficial practice forecasts for the October meeting. A handful of live forecasts is a demonstration, not evidence.

---

## ⚠️ Limits

* **Small effective sample:** 63 rate moves in the test sample and 10–15 genuine turning points; p-values are descriptive.
* **The market benchmark is a yield-curve proxy, not fed funds futures.** Every claim here is "beyond a yield-curve proxy", never "beyond the market". The proxy does not sharpen as the meeting approaches, which is consistent with yields that span several meetings.
* **Fallback vintages:** some pre-2011 meetings use the first archived vintage for $u^{\ast}$ and jobless claims (flagged and counted above).
* **Fixed $r^{\ast} = 2$** in the Taylor gap; a drifting neutral rate would bias that feature over time.
* **The label ignores the size of the move**; the surprise track is the magnitude-aware test.
* **The surprise predictors are FRED-based approximations** of Bauer and Swanson's (Nasdaq for the S&P 500, WTI for their commodity index; no payroll consensus surprise or Treasury skew).

---

## 📁 Repository Architecture

```text
├── results/
│   ├── PREREGISTRATION_V5.md              # The locked test, secondary hypotheses and procedures
│   ├── PREREG_LOCK.json                   # SHA-256 lock of the specification, written by the first run
│   ├── REPORT_v5_primary_locked.txt       # Full report of the locking run (primary, turning points, exploratory)
│   ├── REPORT_v5_with_surprise_track.txt  # Same data and models, plus the surprise track
│   ├── turning_points_v5.csv              # The 19 out-of-sample turning points, model by model
│   ├── forecast_ledger.jsonl              # Hash-chained record of live forecasts
│   ├── HORIZON_EXPLAINED.md               # Plain-English note: why 11 days, and what happens closer in
│   ├── WHY_NOT_JUST_USE_FEDWATCH.md       # Plain-English note: how futures-based odds work and what this study adds
│   ├── explore/                           # Exploratory reports (loss decomposition, winsorized test, horizons)
│   └── figures/                           # Four story figures with captions (FIGURES.md)
└── README.md
```

---

## 🔒 Code & Data Availability

This repository publishes the **results**. The pipeline code (five Python files: shared engine, research run, forecasting, integrity checks, exploratory analysis) and the cached data are kept in a private repository and are **available on request**.

Everything is built from public primary sources, so the results can be checked independently: FRED and ALFRED (St. Louis Fed) for the macro vintages, policy target and market series; the Federal Reserve's FOMC calendars and historical materials for meeting dates; and the San Francisco Fed's Monetary Policy Surprises data file. The definitions and rules above are complete.

---

## 📚 References

* Bauer, M. D., and E. T. Swanson (2023). "An Alternative Explanation for the 'Fed Information Effect'." *American Economic Review* 113(3): 664–700.
* Bauer, M. D., and E. T. Swanson (2023). "A Reassessment of Monetary Policy Surprises and High-Frequency Identification." *NBER Macroeconomics Annual* 37: 87–155.
* Orphanides, A. (2001). "Monetary Policy Rules Based on Real-Time Data." *American Economic Review* 91(4): 964–985.
* Taylor, J. B. (1993). "Discretion versus Policy Rules in Practice." *Carnegie-Rochester Conference Series on Public Policy* 39: 195–214.
* Diebold, F. X., and R. S. Mariano (1995). "Comparing Predictive Accuracy." *Journal of Business & Economic Statistics* 13(3).
* Clark, T. E., and K. D. West (2007). "Approximately Normal Tests for Equal Predictive Accuracy in Nested Models." *Journal of Econometrics* 138(1).
* Sahm, C. (2019). "Direct Stimulus Payments to Individuals." In *Recession Ready*, The Hamilton Project.
