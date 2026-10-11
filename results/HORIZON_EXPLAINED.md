# Does Waiting for the Last Data Help? The Horizon Result in Plain English

*Written 25 Sep 2026. Explains the exploratory horizon analysis in
`results/explore/20260925-204723-horizon/` (report: `HORIZON_REPORT.md`, figure: `horizon_logloss.png`).
Exploratory: the pre-registered result is unchanged.*

---

## The question

The project's main test makes its forecast **11 days before** each Fed meeting, the last day before
Fed officials go silent (the "blackout"). But some important economic news comes out *after* that
day. For the September 15–16, 2026 meeting, for example, the inflation report (CPI) came out on
September 11, just two business days before the meeting.

So the question was:

> **If we wait and forecast closer to the meeting, after that last news is out, does economic data
> start to help?**

## What we did

We ran exactly the same test again, changing only the forecast day:

| Forecast day | What that usually means (for a Tuesday meeting) |
|---|---|
| **11 days before** | The Friday before the Fed goes silent (the original, locked test) |
| **7 days before** | The Tuesday one week out |
| **4 days before** | The Friday before the meeting, after the week's inflation report and jobless claims |
| **1 day before** | The Monday before the meeting: everything is known |

Everything else stayed identical: the same meetings, the same models, the same scoring and the same
statistical test. We also wrote down the rules for judging the result **before** running it, so
we couldn't move the goalposts afterwards.

Both models were compared at every forecast day:
- **"Yields only":** uses bond market prices plus recent Fed moves.
- **"Yields + economy":** the same, plus four economic clues (inflation, unemployment and jobless claims).

The score is **log loss**: lower is better, and it punishes confident wrong guesses hard.

## What we found

| Forecast day | Yields only | Yields + economy | Is the difference real? |
|---|---|---|---|
| 11 days before | 0.512 | 0.699 | No (p = 0.30) |
| 7 days before | 0.544 | 0.729 | No (p = 0.33) |
| 4 days before | 0.515 | 0.693 | No (p = 0.34) |
| 1 day before | 0.534 | 0.746 | No (p = 0.25) |

*(136 meetings from 2000 to 2026, leaving out the years when rates were stuck at zero. A "p" above
0.10 means the gap could easily be luck.)*

**In one sentence: waiting for the last news doesn't change anything. At every forecast day,
adding economic data made no reliable difference.**

## It isn't because nothing new came in

The later forecast days really did see fresher data. Four days before the meeting, compared with
eleven days before:

- **27%** of meetings had a newer inflation report (CPI)
- **17%** had a newer PCE inflation figure (the Fed's preferred measure)
- **13%** had a newer jobs report
- **100%** had newer weekly jobless claims

The model saw this fresher news and still didn't do better. Whatever the economic data knows, the
bond market seems to know it too, at every point in the final two weeks.

## One surprise

We expected the **yields-only** model to get sharper as the meeting got closer, because traders
have more information and less time left for surprises. It didn't: its score stayed about the same
(0.51 at 11 days, 0.53 the day before).

A likely reason, not yet proven: the bond rates this project uses (3-month bills, 6-month bills and
the 2-year yield) cover **several future meetings at once**, not just the next one. They're a blurry
picture of the next decision. Instruments that price *only the next meeting* (fed funds futures)
would probably sharpen in the final days, but they aren't freely available for the whole history,
so the project doesn't use them. This is why the write-up should always say "**beyond a yield-curve
proxy**", not "beyond the market".

## What it means for the project

1. **The 11-day choice isn't hiding anything.** A fair worry was that the test ignored the last,
   most-watched data. It doesn't matter: the answer is the same with or without it.
2. **The main finding holds up.** Across four different forecast days, economic data adds nothing
   detectable to a yield-curve model for predicting the decision itself.
3. **The interesting finding is elsewhere.** The project's positive result is about *surprises*:
   economic data can partly predict how surprised markets will be when the Fed announces. That
   result also held up from both 11 days and 1 day before.

## Honest caveats

- These extra forecast days are **exploratory**. Only the 11-day test was pre-registered.
- Running several extra tests makes a lucky-looking result more likely, but none appeared anyway.
- Only about 10–15 real turning points exist in the data, so no single test here is very powerful.

---

*One line for the write-up: "Moving the forecast from 11 days to 1 day before the meeting, after
the final CPI and jobs releases, leaves the result unchanged: real-time macro data adds no
detectable information beyond a yield-curve proxy at any horizon (exploratory; p = 0.25–0.34)."*
