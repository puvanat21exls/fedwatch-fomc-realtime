# Why Not Just Use CME FedWatch? What It Does, Why It's Hard to Beat, and What This Project Answers

*Written 10 Oct 2026. A plain-English explainer for the question every reader and interviewer asks.*

---

## How CME FedWatch works

CME FedWatch reads the price of **fed funds futures**: contracts that pay out based on the average
Fed interest rate over a given month. Traders put real money on where that rate will be.

- The price tells you the rate the market expects: roughly **100 minus the price**.
- Example: if the current rate is 4.00% and the futures imply 4.16% for the month after the meeting,
  the market is pricing about a 64% chance of a 0.25% hike, because 0.16 is 64% of the way to 0.25.

So FedWatch is not a forecasting model. It translates what traders are already betting.

## Why it's so hard to beat

1. **It already contains everything public.** When the jobs report comes out at 8:30am, futures
   reprice within seconds. Any model built on public data is using information that is already in
   the price.
2. **Real money makes it self-correcting.** If the price were predictably wrong, someone could profit
   by trading against it, and doing so would push the price back. Easy mistakes get traded away.
3. **The Fed tells markets what it is going to do.** The modern Fed dislikes surprising markets, so
   officials signal decisions in advance through speeches, and futures absorb those signals. By
   meeting day the outcome is often 90%+ priced in.
4. **It targets exactly the right thing.** Each contract is tied to a specific month, so it isolates
   the next meeting very precisely.

A caveat for accuracy: futures prices are not pure probabilities. They include a small risk premium,
so they are slightly biased. But as a forecast of the next decision, they are the benchmark almost
nothing beats.

## What this project answers

The project does not try to beat FedWatch. It asks a narrower question, and the answer has three parts.

**1. For the decision itself, macro data adds nothing detectable.**
At 11 days, 7 days, 4 days and 1 day before the meeting, adding inflation, unemployment and jobless
claims to a market-based model made no reliable difference. (The 11-day test is the pre-registered
one, p = 0.30; the other horizons are exploratory, p = 0.25 to 0.34.)

**2. The market benchmark here was weaker than FedWatch, which makes that result stronger.**
The project uses Treasury yields (3-month, 6-month and 2-year), which blur several meetings together.
Fed funds futures are sharper. If macro data cannot beat this cruder proxy, it almost certainly cannot
beat the real thing. This is why the write-up always says "**beyond a yield-curve proxy**", never
"beyond the market".

**3. But the market's mistakes are not completely random.**
When the Fed announces, markets are sometimes caught off guard. Economic data known beforehand can
partly predict which direction that surprise goes, even 11 days ahead (Clark-West p = 0.008; it held
up when the Covid period was excluded and extreme values were trimmed). Two honest limits:

- The effect is small: a couple of basis points on average.
- It is concentrated in time: about 77% of the evidence was built up before 2009, mostly in the 2001
  and 2007-08 rate-cutting cycles, and one meeting (Dec 2008) is about 27% of it.

This fits what Bauer and Swanson (2023) found: markets tend to underestimate how strongly the Fed
will react to the economy, especially when things are turning.

## In one sentence

**You cannot out-predict the market on what the Fed will do, but when the market is wrong, it is
wrong in a slightly predictable way.**

## The interview answer

*"Markets are very efficient at pricing the decision, so I didn't try to beat FedWatch. I tested
whether real-time macro data adds anything beyond market pricing, and for the decision it doesn't.
What I did find is that markets are not perfectly efficient at pricing the Fed's reaction to data:
the surprises at announcements are partly predictable, mostly around turning points."*

---

*Related: `docs/HORIZON_EXPLAINED.md` (why 11 days, and what happens closer in) and
`results/figures/fig4_surprise_predictability.png` (the surprise finding in one chart).*
