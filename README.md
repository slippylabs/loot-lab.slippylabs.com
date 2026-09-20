# Loot & Drop Lab

Weighted drop tables with nested sub-tables and pity timers, and the real numbers behind them — exact drop rates, expected pulls, chance by pull N, and a Monte Carlo run that agrees. Runs entirely in your browser.

**Live:** <https://loot-lab.slippylabs.com/>

## What it does

- Edit a drop table in place: name, weight, quantity range, value, and nested sub-tables to any sensible depth.
- Three presets — an ARPG chest, a gacha banner, and a three-level nested table.
- A pity timer: hard pity (guaranteed on pull N) and soft pity (the rate ramps from there).
- Exact per-pull chance, mean, median, 90th percentile, chance by pull N, expected value and standard deviation per pull, and the *effective* rate a pity timer actually produces.
- "How many will I have after N pulls" — exact, as a renewal process rather than a binomial.
- A cumulative-probability chart with the 50% and 90% marks, and a Monte Carlo run plotted on top of it.

## How it works

A table is a list of entries with weights; an entry is either an item or another table, and the probability of reaching a leaf is the product of the probabilities along the way.

A pity timer makes the per-pull chance depend on how long it has been since the last hit, so `rates[k]` is the chance of hitting on the k-th pull since the last one — the base rate below the soft pity, then a straight line to a certainty at the hard pity. From that array everything else follows: `P(first hit on pull k) = rates[k]·Π(1 − rates[j])` for j < k, and the mean, median and cumulative curve are sums over it.

Because a pity timer makes consecutive pulls dependent, "how many copies in N pulls" is **not** binomial. It is computed exactly by carrying the joint distribution over (pity counter, copies so far) forward one pull at a time. The expected count is accumulated as the sum over pulls of the chance of hitting on that pull, which is exact however low the display cap is set — reading it back off a capped histogram undercounts the tail.

Quantities use the **discrete** uniform variance `((b−a+1)² − 1)/12`, not the continuous `(b−a)²/12`. For a 1–5 drop those are 2.00 and 1.33 — a real difference that compounds over a thousand pulls.

## Verification

Three independent routes to the same numbers:

- **Exact rational arithmetic.** `verify_loot.py` re-derives every leaf probability with Python `Fraction`s — no floating point at all — and the page matches to **1e-15**, with each table summing to exactly 1.
- **An absorbing Markov chain.** The pity distribution's mean is checked against the fundamental matrix `(I − Q)⁻¹` solved by numpy inversion, which shares no algebra with the page's forward recurrence. Agreement to **1e-9** over five configurations. A flat rate reproduces the geometric distribution exactly.
- **Monte Carlo.** The copies-in-N-pulls DP agrees with 200,000 simulated runs on the mean *and* the whole distribution; the weighted roller passes a chi-squared test over 400,000 rolls down the three-level nested table; and the exported `roll()` function, run in Node, reproduces the same rates.

**270 checks.**
