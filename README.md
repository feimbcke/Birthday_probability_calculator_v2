# Birthday_probability_calculator_v2

What are the odds that **N people out of M share a birthday**? This page answers it two
ways at once: an exact closed-form calculation, and a Monte Carlo simulation you can
watch converge on it.

Open [index.html](index.html) in a browser. No build step, no dependencies, no network —
one self-contained file.

![88 people, the odds that 3 of them share a birthday: 51.11%, shown with the exact curve across group sizes and a 100,000-trial simulation converging on it](preview.png)

## The generalized birthday problem

The familiar version (N = 2, "some pair shares a day") has a one-line formula. The
general version — *at least N people on the same day* — does too, and it is exact.

Let D = 365 days and M people. The number of ways to assign M people to days so that
**no day holds N or more** is the M-th coefficient of a truncated exponential series,
raised to the number of days:

```
P(no day has ≥ N) = M! / D^M · [x^M] ( 1 + x + x²/2! + … + x^(N-1)/(N-1)! )^D
```

The page multiplies that polynomial out directly (exponentiation by squaring, truncated
at degree M). Every coefficient is positive, so nothing cancels; normalising by the
largest coefficient after each multiply and carrying the log of the scale factor keeps
it stable to ~14 significant digits. One evaluation yields the probability for *every*
group size at once, which is what draws the curve.

It runs in about a millisecond. Verified against the known thresholds for a 50% chance:

| People sharing a day | Group size needed |
|---|---|
| 2 | 23 |
| 3 | 88 |
| 4 | 187 |
| 5 | 313 |

## So why simulate at all?

Because the simulation shows what the formula hides — how long a random experiment
takes to agree, and how wide the error stays until it does. The convergence panel plots
the running estimate against the exact answer inside a 95% band, on a log scale. Error
falls with the square root of the trial count, so each extra digit of precision costs a
hundred times the work. At 1,000 trials the estimate is still worth about ±3 points.

Trials run in chunks on `requestAnimationFrame`, so the page stays responsive and the
run can be stopped mid-flight; the partial result is still a valid estimate.

## What's on the page

- **The question is the control** — edit the numbers in the headline sentence directly.
- **A year strip** — 365 columns, one per day, showing a single random group. Ledger
  lines let you count a column by eye; days that meet the threshold flare and get a marker.
- **The curve** — exact probability against group size, with the crossings for even odds,
  9-in-10 and 99-in-100. "Show the numbers" opens the same data as a table.
- **The simulation panel** — running estimate, exact reference, 95% band, and the gap
  between simulated and exact.

## Assumptions

365 days, birthdays uniform and independent. Real birthdays are neither — there is
seasonal clustering and a dip around Feb 29 — which makes real-world coincidences
slightly *more* likely than these figures. Leap days are not modelled.
