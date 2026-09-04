# Risk Quantifier

From a five-by-five matrix to a loss distribution. Place up to five risks on a likelihood-by-impact
grid, give each the frequency and loss range its cell was standing in for, and read what ten
thousand simulated years say about each of them, and about all of them together.

**Live:** https://rootcawsllc.github.io/risk-quantifier/

![Two risks placed on the matrix, which sits in the left-hand panel with muted low, medium and high tints. On the right, each risk has a card with its cell, a starting-point picker and low/likely/high fields for frequency and loss per event; the first has been loaded from a UK financial-services data-breach benchmark and shows its badges, a "not good for" caveat and six cited sources with their limitations. Below, a ledger compares what the matrix said with what the model says, one row per risk with a small histogram, a typical year, an average, a bad year and the share of loss-free years; the portfolio total is withheld because the two risks are priced in different currencies](preview.png)

## What it does

- **The matrix is the control.** Tap a cell to place a risk, tap it again to remove it. Each cell
  hands the risk an illustrative starting range for frequency and for loss per event, so placing
  a risk gives the model something to run on straight away.
- **Say what the cell meant.** Every placed risk gets a card with a frequency in events per year
  and a loss per event, each as a low, likely and high. Or pick a source-backed benchmark and
  replace the illustrative range with one that cites a public source for every parameter, carries
  that source's stated limitation, sets the currency, and says what it is *not* good for.
- **Live results.** Ten thousand simulated years per risk, recomputed whenever anything changes.
  A ledger shows one row per risk: the cell it came from, a small histogram of the years, the
  typical year (median), the average, the bad year (1 in 10) and the share of loss-free years.
  With more than one risk in the same currency, a final row shows the portfolio total, summed year
  by year before sorting. Mixed currencies get a note instead of a total.
- **The comparison is the point.** Above the ledger: how many cells and how many colours the
  matrix had to offer, against how many distributions came out. A cell is a position and a tint
  shared with every other risk in the same band; a row is an amount.

## How it works

Each risk starts life as a click, and the cell you click sets a real starting range, not a
placeholder: "Rare / Negligible" and "Almost certain / Severe" produce genuinely different frequency
and loss ranges, not the same three numbers with a different colour. That is the point of routing
the matrix into a simulation instead of stopping at it.

Those cell ranges are still invented, which is the honest weakness of every tool like this. So each
risk can instead be loaded from a benchmark: a governed shard for a country, sector and threat
where every parameter traces to a named public source. Picking one replaces the range, sets the
currency, and shows the citations behind it alongside the shard's own statement of what it will not
support. A range with no provenance and a range with six sourced parameters should not look equally
authoritative, and here they do not.

From there, each simulated year:

1. **Frequency** is drawn from a PERT distribution over the low, likely and high. PERT rather than
   a plain triangle because it weights the likely point instead of treating it as no more probable
   than the edges.
2. That frequency becomes the rate of a **Poisson draw**, which produces the actual event count for
   the year. A frequency of three a year does not mean exactly three: most years will be near it,
   some will be zero, a few will be considerably more.
3. **Every event that year gets its own independent loss draw**, also PERT, and the year's total is
   the sum of them.

That last step is the one that matters most. The common shortcut is `total = events × one loss
draw`, treating a year with three events as a year with one event that cost three times as much.
Those are different distributions: the shortcut suppresses variance and understates the tail,
because the tail is exactly the case where several bad things land in the same year.

When more than one risk is placed, the portfolio total is the year-by-year **sum** of every risk's
simulated losses, not the sum of their individual percentiles, which is a different and larger
error. Percentiles do not add.

## Deployment

A standalone HTML file with no dependencies and no build step. The benchmark data is inlined, so
the file works opened straight from the filesystem as well as from any web server. Deployed to
GitHub Pages at https://rootcawsllc.github.io/risk-quantifier/.

## Honest limits

- **Per-event losses are drawn fully independently within a year.** Real losses are rarely that
  clean: a breach severe enough to trigger a large notification cost is also more likely to
  trigger a large liability claim. This tool does not model that correlation; yeetmap's priced
  loss modules do, via shared per-event latents.
- **The starting range for each cell is illustrative, not calibrated.** It exists so a click
  produces a real, distinct range instead of an arbitrary flat one. Adjust it, or load a
  benchmark, before treating any output as an estimate for your organisation.
- **The benchmarks are starting points, not benchmark-grade figures.** Each shard carries its own
  status, and most are governed starters rather than reviewed benchmarks. Several borrow a
  frequency from another country because no local per-firm rate is published, and say so in their
  own caveat. Read the per-parameter limitations before quoting any of it.
- **Benchmark data is a snapshot.** The figures are baked into `index.html` from one upstream
  commit and do not update themselves; regenerate when the shards change.
- **No FX conversion.** Shards are priced in their own currency and there is no rate table, which
  is why a mixed-currency portfolio shows no total rather than a converted one.
- **No controls model, no reproducible seeding, and no provenance for anything typed by hand.**
  Benchmark parameters carry their sources; values you enter yourself carry nothing. This is a
  single-file teaching tool, not an audited engine, and it has no tests. For the real thing, with
  FAIR-CAM controls, an inverse pass, AI-assisted extraction with a strict tool schema and a full
  test suite, see yeetmap.
- **Not a substitute for a real risk assessment.** It exists to make one point vivid: a cell on a
  matrix and a loss distribution are not the same kind of answer, and only one of them supports
  arithmetic.

## Attribution

Benchmark shards come from [RiskShard](https://github.com/raviaxo/RiskShard) by
[raviaxo](https://github.com/raviaxo), an evidence-governed cyber risk quantification project,
AGPL-3.0, where every parameter traces to a reviewed public source. The frequency and impact
ranges, source citations, confidence levels and "not good for" statements in this tool are
RiskShard's, carried through unchanged; the picker, the matrix and the simulation are not. Each
shard's underlying sources are credited individually in the tool itself and remain the property of
their publishers.

Uses the same compound-Poisson approach as yeetmap, this author's full FAIR (Factor Analysis of
Information Risk) quantification engine: frequency as a Poisson rate, independent per-event
magnitudes summed, not multiplied. FAIR itself is a model published by the
[FAIR Institute](https://www.fairinstitute.org/); this tool does not implement or reproduce
FAIR's published standards, only the same underlying simulation principle.

## License

Copyright (c) 2026 RootCaws LLC.

[GNU AGPL v3 or later](LICENSE). If you modify this and run it as a network service, the AGPL requires you to offer your users the modified source under the same terms.
