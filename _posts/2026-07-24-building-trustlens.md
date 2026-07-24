---
title: "Building TrustLens: Author-Weighted Sentiment for Stock Market Prediction"
date: 2026-07-24
categories:
  - capstone
tags:
  - machine-learning
  - finance
  - nlp
  - capstone
---

## Introduction

I am Abhishek Kalhan, a Master's student in Computer Science at Full Sail University and a Customer Success Engineer at Quartic.ai. My focus is transitioning into AI Engineering, and my capstone project sits at the intersection of machine learning, financial data, and social media analysis.

My capstone is called TrustLens. It is a research pipeline that predicts next-day stock price direction using StockTwits sentiment, weighted by each author's historical accuracy. The idea is simple. Every existing sentiment tool treats all posts equally. But some posters have a real track record and others do not. TrustLens weights the signal by who is actually right over time.

![Comparison of TrustLens with prior work](/assets/images/trustlens-dashboard.png)

## What I worked on this month

This month I completed the proposal phase of the capstone. That included finalizing the research question, benchmarking against prior published work, building the evaluation design, and presenting to my capstone committee.

The core pipeline has four parts:

1. Author trust scoring computed from historical bullish and bearish call accuracy.
2. A weighted signal aggregated per ticker per trading day.
3. A sequence model (LSTM or Transformer) that takes the recent weighted signal and outputs a next-day directional prediction.
4. A prospective forward test on live 2026-present data.

![Comparison of TrustLens with prior work](/assets/images/trustlens-walkforward.png)


Tools I used this month:

- Python and PyTorch for the modeling pipeline
- SQLite for storing StockTwits data
- Weights and Biases for experiment tracking
- The JQD Archive as the historical StockTwits dataset (550 million posts, 2008 to 2022)
- StockTwits public API for the live forward test data

I also spent significant time on the literature review. I benchmarked TrustLens against four closest prior papers: Bouadjenek et al. (2023), Zhou et al. (2025), Xu and Keselj (2019), and Divernois and Filipović (2024). Getting the citations right and understanding exactly how my approach differs from each was more work than I expected.

![Comparison of TrustLens with prior work](/assets/images/trustlens-comparison-table.png)

## Challenges

The biggest challenge this month was scoping. My first version of TrustLens was too ambitious. It included fine-grained emotion classification, hand-tuned trust formulas, and multiple parallel hypotheses. Prof. Marpaung pushed back on the scope, and I ended up rewriting the proposal to a single testable hypothesis with a defensible evaluation design. That rework was hard but the project is much stronger for it.

I also had to be honest about novelty. When I did a proper literature review, I found that Zhou et al. (2025) had already demonstrated the core mechanism of trust-weighted sentiment for stock prediction. Rather than pivoting away, I reframed TrustLens as a rigorous validation and extension: full walk-forward across multiple market regimes and a genuinely prospective forward test, both of which Zhou did not do. The mechanism is a refinement of prior work. The rigor of the evaluation is the primary contribution.

## Retrospective

**What went right this month:**

- The proposal defense went well overall. The core mechanism and evaluation design were received. I was able to defend my choices with references from the literature and specific accuracy numbers.
- The literature review process caught real citation errors and clarified how my work fits the landscape.
- The reduced scope strategy that worked for Milestone 1 worked again for Milestone 2. Deliberately scoping down instead of overcommitting has been the approach that gets things submitted.

**Suggested Improvements:**

- I underestimated the time required for the literature review. I had to verify each paper's methodology carefully and correct citations that had propagated through earlier drafts.
- I was working with a scope (four highly traded stocks) that the committee found too narrow. The external validity concern was a legitimate one that I did not fully anticipate.

**How I can improve moving forward:**

- Address the committee's feedback head-on: expand the ticker set to a sector-stratified sample, incorporate technical features alongside sentiment, and expand the literature review to include multi-factor prediction and sector-conditional sentiment work.
- Draft a research ethics section before the next milestone.
- Confirm each direction change with Prof. Marpaung before writing.

## What I have learned

The biggest lesson from this month is that rigor of evaluation matters more than novelty of mechanism. Prior work in this space has been mostly retrospective, and even careful retrospective evaluations can leak information subtly. A prospective forward test on data that literally did not exist when the model was frozen is a much stronger claim, even when the underlying mechanism is a refinement rather than an invention.

## Ready for the next phase

Yes. The proposal is submitted, the committee's feedback is documented, and the next phase is implementation with the revised scope. I plan to work through the committee's recommendations with Prof. Marpaung and adjust before the development freeze at the end of September.