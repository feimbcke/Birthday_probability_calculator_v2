# Birthday_probability_calculator_v2

What are the odds that **at least N people out of M share a birthday**? This page answers it
two ways at once: an exact closed-form calculation, and a Monte Carlo simulation you can
watch converge on it.

Open [index.html](index.html) in a browser. No build step, no dependencies, no network —
one self-contained file.

![23 people, the odds that at least 2 share a birthday: 50.73%, shown with the exact curve across group sizes and a 100,000-trial simulation converging on it](preview.png)

Edit either number in the headline and everything recomputes as you type; the simulation
re-runs on its own shortly after you stop.

## The generalized birthday problem

The familiar version (N = 2, "some pair shares a day") has a one-line formula. The
general version — *at least N people on the same day* — does too, and it is exact.

Let D = 365 days and M people. Counting directly is awkward because a group can clash on
several days at once, so count the complement: arrangements where **no** day holds N or
more. A single day receiving fewer than N people contributes the truncated exponential
series, and the 365 days are independent, so the whole year is that series to the 365th:

```
P(no day has ≥ N) = M! / D^M · [x^M] ( 1 + x + x²/2! + … + x^(N-1)/(N-1)! )^D
```

`[x^M]` is the coefficient of x^M. The page multiplies that polynomial out by repeated
squaring — about log₂(365) ≈ 9 multiplications rather than 365.

At N = 2 the series collapses to `(1 + x)`, giving back the familiar
`365 · 364 · … · (365−M+1) / 365^M`.

### The numerical catch

Raised to the 365th power, those coefficients span hundreds of orders of magnitude and the
smallest ones flush to zero in floating point — which silently reads as "collision
certain" when it is nothing of the kind. (An earlier version of this file had exactly that
bug: it reported 100% for thresholds of 4 and 5 past about 1,000 people.)

The fix is to **tilt** the series before raising it: rescale it into a genuine probability
distribution (a truncated Poisson, with the tilt chosen so its mean lands where the
interesting group sizes are). The coefficients then become `P(365 tilted days hold m
people in total)` — all within [0, 1], summing to one, and largest exactly where accuracy
matters. The tilt is undone afterwards in logarithms, so the answer is unchanged; only the
arithmetic is made safe.

Verified against the known sequence for the smallest group giving even odds
([OEIS A014088](https://oeis.org/A014088)):

| At least N sharing | Group size needed |
|---|---|
| 2 | 23 |
| 3 | 88 |
| 4 | 187 |
| 5 | 313 |
| 6 | 460 |
| 7 | 623 |
| 8 | 798 |
| 9 | 985 |
| 10 | 1,181 |

Cross-checked against independent simulation at eight (M, N) pairs; the largest
disagreement was 1.5 standard errors, which is ordinary sampling noise.

## Where the exact method stops

The page computes exactly up to **N = 10** and switches to simulation above that. This is a
practical budget, **not** a mathematical wall — the formula is valid for every N.

What runs out is time. The polynomial's degree must reach the group size and the cost grows
with the square of it. Past N = 10 the group sizes where the answer stops being
approximately zero run into the thousands (N = 11 needs ~1,385 people for even odds,
N = 12 needs ~1,596), and the exact pass would be slow enough to stall a page that
recalculates while you type. Sampling scales linearly in the group size, so it takes over.

## Why simulate at all

Because the simulation shows what the formula hides — how long a random experiment takes to
agree, and how wide the error stays until it does. Each trial is Bernoulli(p), so the hit
count is Binomial(Z, p), the estimate is unbiased, and its standard error is
`√(p(1−p)/Z)`. The convergence panel plots the running estimate inside a ±1.96·SE band on a
log scale: error falls like 1/√Z, so each extra digit of precision costs a hundred times
the work.

Trials run in chunks on `requestAnimationFrame`, so the page stays responsive and a run can
be stopped mid-flight; the partial result is still a valid estimate.

## What's on the page

- **The question is the control** — edit the numbers in the headline sentence directly.
- **A year strip** — 365 columns, one per day, showing a single random group. Ledger lines
  let you count a column by eye; days meeting the threshold flare and get a marker.
- **The curve** — exact probability against group size, with the crossings for even odds,
  9-in-10 and 99-in-100, and a table view.
- **The simulation panel** — running estimate, exact reference, 95% band, and the gap
  between simulated and exact.
- **About the math** — a full write-up of both methods, aimed at someone who wants to
  explain them rather than just use them.

## Assumptions

365 days, birthdays uniform and independent. Real birthdays are neither — there is seasonal
clustering and a dip around 29 February — and any departure from uniformity *increases* the
chance of a coincidence, so these figures are mild underestimates of the real world. Leap
days are not modelled.
