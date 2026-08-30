---
title: "Five Ways to Score Trust, and Why the Simplest One Won"
date: 2026-08-30
categories:
  - capstone
tags:
  - machine-learning
  - finance
  - data-engineering
---

## Where things stand

TrustLens predicts next day stock direction from StockTwits posts, weighted by each author's own track record. Last month I wrote about finishing the proposal phase. This month was about turning "each author's track record" into an actual number, and it turned out to be more work than I expected.

## What I worked on this month

### Fixing a silent labeling bug

The main goal for Milestone 5 was building the trust scoring engine: a formula that turns an author's history of bullish and bearish calls into a single trust score. Before I could trust that formula, though, I had to trust the labels it was being tested against.

While sanity checking the rebuilt direction labels, I noticed the spread in up-rate across tickers was unusually wide, about 56 points from the lowest to the highest. That is too wide to be normal market behavior, so I dug in. The cause was a filtering step earlier in the pipeline that dropped any row missing a 200 day moving average. That silently discarded the first 199 trading days for every ticker. For tickers that listed partway through the evaluation window, like Snowflake and QuantumScape, this meant almost every early post got matched to the wrong price row. About 1.4% of all labels across the dataset were wrong as a result, more than half of Snowflake and QuantumScape's labels specifically. I patched both the feature script and the label builder, rebuilt the labels, and added a guard so it cannot happen again silently. After the fix, the up-rate spread dropped to about 20 points, which is a much more believable number.

![Per-ticker up-rate spread before and after the label fix](/assets/images/trustlens-uprate-spread.png)

### Comparing five trust formulas

With clean labels in place, I ran the actual trust formula comparison. The formula I started the month with used per-call exponential decay, but Prof. Marpaung asked for that to be replaced with something based on real inactivity rather than call-by-call decay, so before comparing anything I swapped it for an absence based decay. I then tested that alongside three more candidates: a version that corrects for how bullish biased a ticker is, a formal skill statistic borrowed from meteorology, and a shrinkage estimator that pulls thin data authors toward a neutral score. None of them beat the plain formula, an author's raw hit rate with a minimum call requirement before they count, which separated the best and worst authors by about 1.5 percentage points and beat chance in 9 of 12 walk-forward rounds.

Three of the alternatives are directly comparable on that same decile separation metric, shown in the chart below: the bullish bias correction came close at 1.08 percentage points, and the skill statistic barely cleared zero at 0.11. The other two used different diagnostics. Shrinkage's best setting turned out to be no shrinkage at all, mathematically identical to the plain formula, with every stronger setting doing worse. The absence based decay was swept across 20 configurations and only beat a no-decay baseline in 3 to 8 of 12 walk-forward rounds, close to a coin flip, with the best configuration adding just 0.27 percentage points and the worst losing 0.52. That was not the outcome I expected going in.

![Decile separation: plain hit rate vs. the comparable alternative formulas](/assets/images/trustlens-decile-chart.png)

### Keeping the live data collection running

Separately from the modeling work, the always-on VM that has been quietly collecting live StockTwits data for the eventual 2026 forward test crashed. An uncommon network error was slipping past the exception handler and killing the whole process. Progress was not lost since the database only commits after a full successful page, so I widened the error handling, confirmed the process was healthy again, and merged about 295,000 newly collected rows back into the main database.

![VM status output showing the merge complete](/assets/images/trustlens-vm-status.png)

### Settling the human validation study question

I also worked with Prof. Marpaung on a question that had been sitting open for weeks: whether the project needed a separate human validation study, where volunteers would rate posts alongside the model. We decided it does not. The walk-forward evaluation against real price outcomes stands on its own, which also simplified the ethics paperwork down to a single exemption category since there are no human subjects involved anymore.

### Profiling the label pipeline for COS660

Outside of the capstone milestones directly, this month's COS660 assignment asked for both performance testing and user testing on part of the project. The performance half went well: profiling the label builder with cProfile turned up two things I did not expect. First, the script scans all 106 million rows of the archive every time, even when only a handful of tickers are requested. Second, writing to SQLite accounts for roughly 55% of the total runtime, more than the actual label computation itself. The user testing half is currently stuck. The project's IRB approval only covers secondary research on an existing dataset, and there is no user-facing interface to test in the first place, so the two do not line up. I emailed Prof. Marpaung to flag the mismatch and am waiting to hear back on how to handle that part of the assignment.

Tools used this month: Python and pandas for the pipeline work, SQLite for storage, cProfile and SnakeViz for profiling the label builder, and a GCP free tier VM for continuous background data collection.

## Challenges

The label bug was the hardest thing to catch this month, precisely because nothing errored. The pipeline ran cleanly end to end and produced numbers that looked plausible at first glance. It only became visible because a downstream number looked slightly too extreme to be real, which meant I had to go looking for a bug I did not know existed. That is a useful lesson: a script finishing without errors is not the same as a script being correct, and it is worth building in a sanity check step at each stage instead of only at the end.

The formula comparison was also more time consuming than I planned for. My working rule for this project is that every threshold and design choice has to be backed by a measurement rather than a guess, which meant I could not just pick a reasonable sounding formula and move on. I had to actually build and test all five alternatives before I could defend sticking with the simplest one.

The COS660 assignment surfaced a structural problem rather than a technical one. The project's scope changed enough since the IRB was first approved that half of a required assignment no longer fits the project as it currently exists. That is not something I can fix by working harder on it, only by getting a decision from my advisor, which is a different kind of waiting than debugging.

## Retrospective

**What went right this month:**

- I caught the labeling bug before it propagated into the trust scores and, eventually, the model results. Finding it now instead of at the defense saved a lot of pain.
- I ran a genuine, disciplined comparison across five formulas instead of settling on the first one that seemed reasonable. Every number in the current design traces back to a measurement.
- The VM crash got fixed without losing any collected data, because of how the database commits were structured.

**What went wrong this month:**

- The formula sweep took longer than I budgeted for, which pushed the milestone submission closer to the deadline than I would have liked.
- Two of the twelve walk-forward evaluation rounds come back negative under every formula I tested. I have not root caused this yet. Both sit near periods when the market shifted regimes, so that is my working theory, but it is still open.
- The user testing half of the COS660 assignment is stuck on an IRB scope mismatch I cannot resolve myself, and I am waiting on my advisor's response.

**How I can improve moving forward:**

- Build a quick sanity check into every pipeline stage as I write it, rather than trusting a script because it ran without errors.
- Keep a running list of honest limitations as I find them, instead of collecting them right before a milestone is due.

## What I learned

The biggest lesson this month was that the simplest formula winning is itself a finding, not a disappointing result. It would have been easy to assume that adding recency weighting or bias correction would obviously help and skip testing the plain version carefully. Testing all of them showed that the signal in this data is small enough that the more complicated formulas were mostly fitting noise. That is worth stating plainly in the thesis rather than glossing over.

This also connects back to the performance and testing material from earlier coursework. Profiling the label builder this month showed that most of its runtime was going into database writes rather than the actual computation, which was a good reminder that the slow part of a pipeline is rarely where you assume it is until you actually measure it.

## Ready for the next phase

Yes. Labels are clean, the trust formula is settled and agreed with Prof. Marpaung, and Milestone 6 is next: building and validating the sequence model that actually makes the predictions.
