# FedWatch v5 — Pre-registration

**Author:** Puvanat  **Drafted:** 22 Sep 2026  **Status:** commit to git *before* the first real-data run of `fomc_trainer_v5.py`
**Machine lock:** the first run writes `Result_v5/PREREG_LOCK.json` with a SHA-256 of everything that can move the primary result (`fomc_core_v5.prereg_spec()`). Later runs report whether they still match.

**Pre-registration commit hash:** 6c6a937c9c5c0a01deb887a51217ec21a021a23d (2026-09-23)  **Lock hash (first 12 chars):** c8b679059019 (locked 2026-09-24T06:06:23Z, run 20260923-230623-train-0b88a7)

**Folder note (25 Sep 2026):** the project was reorganised into `code/`, `data/`, `docs/`, `results/` and `forecasts/`. Files were moved, not changed (every moved file is byte-identical), and the lock hash is unchanged. Old → new: `Result_v5/` → `results/`, `live_log_v5/` → `forecasts/`, `data_cache_v5/` → `data/cache_research/`, `data_cache_v5_live/` → `data/cache_live/`, `external/` → `data/external/`, `fomc_meetings.csv` → `data/`, `fomc_*_v5.py` → `code/`. Earlier records (manifests, run reports, commit messages) keep the old names.

## Checklist before the first real-data run

- [x] Calendar verified against the Fed's pages (`fomc_checks_v5.py calendar`) and mismatches fixed
- [x] `selftest` passes
- [x] This file and the code committed to git (fill in the commit hash above)
- [x] No real-data results have been computed with v5

---

## Primary hypothesis (H1) — the only headline test

**Question:** At the anchor date (11 days before a scheduled FOMC meeting), do compact macro variables measured in real time add information about the decision (Cut/Hold/Hike) beyond a yield-curve market proxy and the policy state?

| Item | Specification |
|---|---|
| Rows | Scheduled meetings from 1994-02-01; features as of D = start − 11 days, close of D |
| Label | Change in target upper bound (day before start → decision + 3 days); ±0.10pp |
| Data | ALFRED vintage in force on D; release-rule fallback only before a series' first vintage (flagged) |
| Control | L2 logit (C = 0.3), features: `chg_since_prev_mtg, chg_last3_mtgs, days_since_move, at_zlb, bill3m_gap, bill6m_gap, y2_gap` |
| Treatment | Same + `taylor_gap, infl_mom, sahm, claims_chg_3m` |
| Common sample | Rows with all `ALL_FEATURES` non-missing |
| Evaluation | Expanding walk-forward, first training ≥ 48 meetings, refit every 8 |
| Test sample | Out-of-sample meetings with target upper bound > 0.25 on D (ex-ZLB) |
| Metric | Multiclass log loss, probabilities floored at 0.001 |
| Test | Diebold–Mariano on per-meeting loss differences, Newey–West 4 lags, two-sided, α = 0.10 |
| Decision rule | p < 0.10 and treatment better → **macro adds information**; p < 0.10 and worse → **macro hurts out of sample**; otherwise **no detectable difference** |

Also reported, but not decisive: the same test on all OOS meetings, including ZLB; and the circular block-bootstrap 90% CI (block 8, 5,000 reps, seed 20260922).

The primary result gets reported in the write-up **whatever it shows**.

## Secondary hypotheses

**S1 — Turning points.** Episodes are OOS scheduled meetings that are one of:
- a **reversal**: a move opposite to the last target change
- a **restart**: a same-direction move after ≥ 4 consecutive holds
- a **pause**: a hold after ≥ 2 same-direction moves

Per episode, compare P(actual outcome) at the turn for treatment vs control (tie if |diff| ≤ 0.005), and lead time (thresholds: 0.25 for moves, 0.50 for pauses). Summary statistic: sign test across episodes. Reported as case studies. Unscheduled turns (e.g. 3 Jan 2001) are listed, not scored.

**S2a — Surprise predictability, replication.**
- **Target:** raw Bauer–Swanson MPS (SF Fed file) matched to scheduled decision dates.
- **Predictors, known the day before the meeting starts:** `payems_yoy, eq_ret_3m, slope_chg_3m, oil_ret_3m, infl_mom, sahm, claims_chg_3m`.
- **Test:** expanding-window OLS from observation 40, vs a zero forecast. Clark–West one-sided, α = 0.10.

**S2b — Surprise predictability, extension.** Same as S2a, with predictors as of the 11-day anchor.

MPS_ORTH is never used as a target.

## Exploratory (never headlined)

All other model × feature-set scores, ordered logit, RF, XGB, ensemble, rich macro set, encompassing regressions, disagreement tables, the revised-vs-real-time rerun (4f), and anything added after the lock.

## Deviations log

| Date | What changed | Why | Lock hash before → after |
|---|---|---|---|
| | | | |

## Official live forecasts

Run `python code/fomc_inference_v5.py --official --asof <anchor> --meeting <meeting start>` **after** the anchor day, once FRED has posted the anchor-day market closes. Then commit and push before the meeting.

**Procedure note (24 Sep 2026, before any official forecast).** Training uses the information public by the *close* of D, and that includes D's Treasury, VIX, Nasdaq and oil closes. FRED posts those closes the next business day. A run *on* D would therefore use the closes of D−1. The official forecast is instead made after D with `--asof D`. It downloads fresh data but keeps only what was public by the close of D, using the same code as training. The script refuses to run until the anchor-day close is available. This changes when the forecast is run, not what is forecast: the pre-registered spec and its lock hash (`c8b679059019`) are unchanged. Code: commit `4c7af1e`.

| Meeting | Anchor (D) | Run on (earliest) | Committed? | Outcome |
|---|---|---|---|---|
| Oct 27–28, 2026 | Fri Oct 16, 2026 | Mon Oct 19 (late afternoon ET) | | |
| Dec 8–9, 2026 | Fri Nov 27, 2026 | Mon Nov 30 (late afternoon ET) | | |
| Jan 26–27, 2027 | Fri Jan 15, 2027 | Tue Jan 19 (Jan 18 is a holiday) | | |
| Mar 16–17, 2027 | Fri Mar 5, 2027 | Mon Mar 8 (late afternoon ET) | | |

## Live model updates

*Rule set 24 Sep 2026, before any official forecast.*

1. **The research result is frozen.** The primary test, S1 and the exploratory tables in the write-up come from run `20260923-230623-train-0b88a7`. S2 comes from `20260923-231425-train-5f7981`. Both use data downloaded on 23 Sep 2026 (`data/cache_research/`; input hashes in each run's `manifest.json`). No later run replaces them.
2. **The live model is retrained between meetings.** Once the previous decision is observable on FRED (about 3–5 days after it) and before the next official run, run `python code/fomc_trainer_v5.py --live-update`. It uses the same code and the same locked spec, downloads into its own cache (`data/cache_update/`) and never touches `data/cache_research/`. It must print `matches the lock`. If it prints `DEVIATION`, there is no official forecast until that is resolved.
3. **Update runs are not the pre-registered result.** Their run folders are named `...-update-...` and their `REPORT_v5.txt` starts with a "LIVE MODEL UPDATE" banner. Their walk-forward numbers include later meetings and are cited only as "updated through <date>".
4. **The official forecast uses the latest update's model.** The forecast ledger records its `bundle_run_id` and SHA-256.

Next retrain: between about Nov 2, 2026 (Oct 28 outcome on FRED) and Nov 30, 2026 (official run for Dec 8–9).
