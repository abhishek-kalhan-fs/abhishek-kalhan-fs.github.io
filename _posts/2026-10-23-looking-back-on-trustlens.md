---
title: "Looking Back: What TrustLens and the CSMS Program Taught Me"
date: 2026-10-23
categories:
  - capstone
tags:
  - reflection
  - TrustLens
  - machine-learning
  - walk-forward testing
---

*By Abhishek Kalhan, October 23, 2026*

## Where things stand

This is my last post for the Computer Science Master's at Full Sail. I defended TrustLens on October 23, 2026, and graduation is October 29. Looking back at my first post from July, it is a little strange to read. The project I described there is not the project I finished, and I think that is the most useful thing I can say about the whole year.

TrustLens asks one question: if you give more weight to StockTwits authors whose past calls were right, do you get a better read on where a stock is going than if you treat everyone the same? The short answer, after a lot of testing, is yes, but only once the test was big enough to see it, and only on days when both reliable and unreliable authors are posting.

![TrustLens, from proposal to defense](/assets/images/trustlens-journey.png)

## My video reflection

{% include video id="https://drive.google.com/file/d/1us4Csew9D_T5wxNhqc1JEXuZ0tZpo4n6/view?usp=drive_link" provider="google-drive" %}

In the video I talk about what I learned in the program, how it changed the way I build and evaluate software, and where I am going next. The written version below goes course by course.

## First post vs. last post

My first post was written right after the proposal defense. Here is how the plan then compares with what I actually built.

| | July (proposal) | October (final) |
|---|---|---|
| Stocks | 4 (AAPL, TSLA, AMZN, SPY) | 854, tested as the Busy 300 and the Quiet 554 |
| Text signal | 12-class emotion classifier planned | Authors' own bullish and bearish tags |
| Trust score | Recency-weighted, planned | Share of an author's past calls that were right |
| Models | LSTM or Transformer, planned | Logistic regression, random forest, linear SVM, LightGBM (LSTM tested and dropped) |
| Evaluation | 8 walk-forward rounds, planned | 16 quarterly walk-forward rounds, plus a 2024 to 2026 holdout on 8 stocks |
| Result | None yet | On the Busy 300 at 20 days, on the same days, the model with trust scores 59.8% vs 56.1% without it |

Even the data description in my first post was wrong. I wrote "550 million posts, 2008 to 2022". When I actually checked the archive, it was about 106 million tagged posts from June 2010 to December 2022. Small thing, but it says a lot about where I was in July: I was repeating numbers before I had checked them myself.

## How the program changed the way I build software

Before this program I built software the way I did at work as a full stack developer: get the feature working, test that it runs, ship it. TrustLens is not that kind of software. It is a pipeline that produces numbers, and the numbers are the product. A bug does not crash anything. It quietly gives you a wrong answer that looks fine.

So I changed how I build. Every script that produces a new result first reproduces a number I already recorded, and if it cannot, I do not trust anything else it says. I write the rule down before I look at the data, for example the rule for picking the 854 stocks. I keep new experiments in a separate database so the results I already reported can never change by accident. And each milestone got a git tag, so I can always go back to exactly what I submitted.

## How the program changed the way I evaluate software

I used to think of evaluation as "does it work?" Now I ask "compared to what, on data it has never seen, and is the comparison fair?" Three things changed for me.

First, the baseline matters more than the score. A model that is right 60% of the time sounds good until you notice that guessing "down" every day would have done almost as well in 2022. I started putting the simple guesses (a coin flip, always-up, always-down) next to every accuracy I report.

Second, the comparison has to be fair. My first write-up said trust added 3.9 accuracy points. Then I noticed the two models were being tested on different sets of days. When I ran them on exactly the same days, trust still helped, but my best accuracy number turned out to come mostly from which days the model chose to act on. I had to take my own number back.

Third, a "no" can mean two things. On 38 stocks, trust lost to the plain crowd in 12 of 12 quarters. That looked like a failure. When I worked out the smallest difference 38 stocks could detect, it was 0.014 in AUC, more than twice the 0.006 I was actually seeing. My test could never have said yes. That one calculation changed the whole thesis.

## How each course fed into TrustLens

### COS540 Research Approaches in Computer Science

This is where I learned to frame a problem as a question with a hypothesis that can be tested. My TrustLens hypotheses (H1 to H4) follow the same pattern: a clear claim, a clear test, and a result that can come out against you. The biggest lesson I carried forward was to check every citation against the original paper. During the capstone I caught two references in my own drafts that pointed to the wrong paper.

### COS550 Advanced Software Engineering

This course gave me the development life cycle that the capstone runs on. I tracked TrustLens in Jira with weekly sprints and wrote decision records in Confluence before any results existed. My design document went through five versions. Writing decisions down early is what let me defend them later, because I could show the choice was made before I saw the numbers.

### COS560 Data Science

TrustLens is mostly a data project, so this course was used every week. The label pipeline reads more than 100 million posts and matches each one to a price move. In August a filtering step silently mislabeled about 1.4% of the labels. I only caught it because one summary number (the spread of up-days across stocks) looked too wide. The habit of looking at the distribution before trusting a result came from here.

### COS630 Data Visualization and Extended Reality

I did not use the XR side in TrustLens, but the visualization side shaped the dashboard. It is a Streamlit app that replays 2022 so the committee can pick a stock and a date and see what the model would have said. The hard part was not making charts, it was making charts a committee member can read in a few seconds. Every number on screen is labeled with the model that produced it.

### COS570 Advanced Artificial Intelligence

My original plan to classify posts into 12 emotions with a fine-tuned DistilBERT model came from this course. I dropped it during the capstone, because the authors' own bullish and bearish tags were already there and the question about trust did not need emotions to be answered. I also used AI assistants throughout the capstone for drafting and coding. The rule I followed was simple: nothing went into the thesis unless I could trace it to a script and a result file myself.

### COS580 Machine Learning

The models in TrustLens (logistic regression, random forest, linear SVM, LightGBM, and an LSTM) are all things I first met in this course. The most important lesson was about splitting data. For stock data a random train and test split is wrong, because the model gets to see the future. That is why TrustLens uses walk-forward testing: train on everything before a date, test on the next three months, slide forward, repeat. I also learned that F1 can flatter a useless model, so I report AUC and MCC beside it.

### COS590 Human-Computer Interaction

The idea behind TrustLens started in a research paper I wrote in this course on trust-filtered, emotion-aware stock prediction. My first capstone version still used a hand-tuned trust formula, and it was rejected because a hand-tuned formula cannot be evaluated objectively. That pushed me to make trust something measured from real outcomes, which is the version I finished.

### COS640 HCI Application Development

The usability ideas from this course came back when I designed the dashboard for the defense. My users were the committee, so the design question was "what does a professor need to see to believe or doubt the result?" The user-testing assignment in COS660 was harder to apply. My IRB covers only an existing archive with no human participants, and at that point TrustLens had no interface at all, so I did performance testing and documented why user testing was out of scope.

### COS650 Software Project: Research, Planning, and Design

This was the proposal month: the written proposal, the design document, the IRB, a SWOT analysis and the proposal defense. The committee said four big tech stocks were too narrow and asked for sector coverage and technical indicators. That feedback is why TrustLens moved to 38 stocks across 8 sectors, picked by a fixed rule, with moving averages, RSI and MACD as inputs.

### COS660 Software Project: Development I

This month built the pipeline: labels, technical features, and the trust score. I tested five trust formulas and the simplest one won, the share of past calls that were right. For the performance assignment I profiled the label builder and found that SQLite writes took about 55% of its runtime, more than reading the files.

### COS670 Software Project: Development II

This was the hardest month and the one I learned the most from. Trust lost on 38 stocks, I tested five published methods on my own data, and then the power calculation showed the test was too small. With my advisor's approval I expanded to 854 stocks, and the result turned around. I also profiled memory and cut the peak of the signal build from 2.67 GB to 0.80 GB with the same output.

### COS680 Software Project: Deployment and Professional Presentation

This month was the hosted dashboard, the thesis and the defense. The school's GitHub organization could not connect to Streamlit Community Cloud, so I deployed from a separate repository on my own account. The thesis became a 16-page IEEE-format paper, and the defense deck went through several drafts after each advisor meeting. The main change from advisor feedback was to spend less time on the journey and more on the results.

## What was hardest, and what would help in the curriculum

**Statistical power.** The biggest time loss in my project came from not knowing early enough how to ask "is my sample big enough to see this effect?" I spent weeks trying new trust formulas on a sample that could never have shown a difference. A short unit on power and sample size before the capstone would have saved me most of August and half of September.

**Testing on time series.** Walk-forward testing and data leakage were things I learned on my own during the capstone. A machine learning assignment that uses time-ordered data and punishes a random split would prepare students much better for any project with dates in it.

**User testing for research projects.** The user-testing assignment assumes an interactive app, and the reference material pointed to game projects. For a research pipeline with an exempt IRB, there should be a clear alternative path, for example evaluating a results dashboard or a structured expert review.

**Milestone mapping and deployment.** Early on, my own design document numbered milestones differently from the graded course milestones, which caused confusion about scope. A simple template that maps the course milestones to the project plan would help. A short guide on deploying from the school GitHub organization would also have saved me an evening.

## Feedback and instruction

The feedback I got from my advisor, Prof. Marpaung, was timely and changed what I did each week. Every weekly check-in ended with something concrete: test the published methods, try more models, approve the expansion, show everything tried and not only the best result. The committee feedback at the proposal defense was also specific enough to act on directly. The course material for the capstone months gave a clear structure, and the weekly milestone rhythm kept the project moving even when the results were not going my way.

## Advice for students starting the CSMS or the capstone

- Pick a project you can explain in two sentences to someone outside your field. You will have to.
- Before you run your main test, work out whether your sample is big enough to answer the question.
- Write down your rules and decisions before you see results, and keep them somewhere your advisor can read.
- Make every new script reproduce a number you already trust before you believe anything new it says.
- Report what failed. My negative results on 38 stocks became the part of the thesis that explained everything after it.
- Schedule your proposal and defense early. You need at least three committee members on the same slot, and that takes longer than you think.
- Write these blog posts as you go. Reading my July post back is the best measure I have of how much changed.

## Where I go from here

My next goal is to move from customer success into an AI engineering role. TrustLens is the strongest piece in my portfolio because it shows the part of AI work I now care about most: building something that produces numbers and then proving those numbers are real. Prof. Marpaung and I have also talked about turning the work into a conference paper in 2027.

There is still research left in TrustLens. One of my findings is that posts made after the market closes carry a good part of the crowd's short-term signal. I could only measure that on my small 2024 to 2026 set of 8 stocks, because the main archive has dates but not times. Testing it at scale needs a larger dataset with timestamps. The other open question is stocks that are always quiet, where few authors have a long enough track record to rate.

## Retrospective

### What went right

- I tested before I argued, and every claim in the thesis traces to a script and a result file.
- Asking "could my test even see this?" turned a dead end into the main result.
- I kept the failed results in the thesis, which made the final result easier to believe.

### What went wrong

- I should have done the power calculation in August.
- I opened my held-back 2024 to 2026 data earlier than my plan said. I reported it as a deviation, but I would rather not have needed to.
- I shared the 3.9-point result before checking that both models were tested on the same days.

## What I learned

The program taught me the tools: machine learning, data science, visualization, research methods. The capstone taught me what to do when the tools give you an answer you did not want. The most valuable skill I am leaving with is not a model or a library. It is the habit of asking whether a result is real before I celebrate it or throw it away.
