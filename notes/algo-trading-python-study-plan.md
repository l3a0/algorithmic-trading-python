# Algorithmic Trading Using Python — Accelerated Study Plan

**Course:** [freeCodeCamp - Algorithmic Trading Using Python](https://www.youtube.com/watch?v=xfzGZB4HhEE) (4h 33m)
**Instructor:** Nick McCullum
**Code repo:** [github.com/nickmccullum/algorithmic-trading-python](https://github.com/nickmccullum/algorithmic-trading-python)
**Your profile:** Advanced Python · Concept mastery + project building · 2+ hour sessions

> **Note:** This course uses IEX Cloud for API data. IEX Cloud's free tier has changed since the course was published (2020). You may need to adapt the API calls or use a sandbox/alternative data source. The *strategies and code patterns* remain fully relevant.

---

## The Big Picture (ELI5)

Imagine you're a chef who has to cook 500 dishes. You *could* taste-test each dish and decide the recipe one at a time. Or you could write a recipe book with rules ("if ingredient X is cheap, buy more; if dish Y is selling well, make more") and let a robot cook for you. **That's algorithmic trading** — writing rules so a computer makes investment decisions instead of you doing it manually, one stock at a time.

This course builds three "recipe books":

1. **Equal-weight index fund** — buy everything on the menu in equal portions
2. **Momentum strategy** — double down on dishes that are selling well
3. **Value strategy** — find underpriced ingredients before others notice

---

## 3 Concepts to Prioritize First

1. **API mechanics** — Every strategy starts with getting data. If you can't pull and parse financial data, nothing else works.
2. **The difference between equal-weight, momentum, and value** — These aren't just coding exercises; they're three fundamentally different *philosophies* about what makes a good investment. Understanding the "why" behind each one matters more than the code.
3. **Batch API calls** — The S&P 500 has ~500 stocks. Making 500 individual API calls is slow and wasteful. Learning to batch calls is a pattern you'll reuse in any data pipeline.

---

## Day-by-Day Study Schedule (4 Days)

### DAY 1 — Foundations + Equal-Weight Index Fund (Part 1)

**Time:** ~2.5 hours | **Video:** 0:00:00 → ~1:00:00

#### Session 1A: Fundamentals & API Basics (0:00 → 17:20) — 17 min watch

**What you're learning:**

- What algorithmic trading is and isn't
- Setting up the IEX Cloud API
- Making your first API call in Python

**The analogy:** This is like learning to turn on the oven and read a thermometer before cooking. Not glamorous, but skip it and you burn the kitchen down.

**Do this:**

1. Watch at 1.5x speed (you're advanced Python — this is setup)
2. Clone the repo: `git clone https://github.com/nickmccullum/algorithmic-trading-python`
3. Set up your environment: `pip install pandas numpy requests xlsxwriter scipy`
4. Get your IEX Cloud API key (or plan your alternative data source)
5. Make your first successful API call before moving on

**Checkpoint question:** *Can you explain to someone else what a REST API returns for a stock quote, and what the JSON structure looks like?* If yes, move on.

---

#### Session 1B: Equal-Weight S&P 500 Index Fund (17:20 → ~1:00:00) — ~43 min watch

**What you're learning:**

- Reading a list of S&P 500 tickers
- Pulling market cap and price data for each
- Calculating equal-weight position sizes
- Building a pandas DataFrame to hold it all

**The analogy:** Regular S&P 500 index funds are *market-cap weighted* — Apple gets a huge slice because it's huge. An equal-weight fund is like splitting a pizza into 500 perfectly equal slices. You're betting that the little guys will catch up to the big guys over time.

**First principles reasoning:**

- Why equal-weight? Market-cap weighting concentrates risk in the biggest companies. Equal-weighting diversifies more broadly but requires more frequent rebalancing.
- The code pattern: `for each stock → get data → store in DataFrame → calculate shares` is a template you'll reuse for every strategy.

**Do this:**

1. Watch at 1.25x, pause when he starts building the DataFrame
2. Code along — type it yourself, don't copy-paste
3. When you hit the "one API call per stock" problem, **pause the video** and try to solve it yourself for 5 minutes before watching his solution

**Before stopping for the day, write down from memory:**

- What data fields you need per stock (price, market cap, number of shares to buy)
- Why making 500 individual API calls is a problem

---

### DAY 2 — Equal-Weight Fund (Part 2) + Momentum Strategy (Part 1)

**Time:** ~2.5 hours | **Video:** ~1:00:00 → ~2:15:00

#### Session 2A: Finish Equal-Weight Fund (→ 1:38:44) — ~39 min watch

**What you're learning:**

- Batch API calls (the key optimization)
- Formatting output into Excel with xlsxwriter
- Taking portfolio size as user input

**The pattern to remember:** `chunks of 100 tickers → batch API call → parse response → append to DataFrame`. This chunking pattern is universal in data engineering.

**Do this:**

1. Watch at 1.25x, code along
2. After finishing, **rebuild the entire project from scratch without the video** — just use your notes. This is the single most effective learning technique. Time yourself.
3. If you get stuck, peek at the code, then close it and keep going

**Spaced repetition pause (15 min):** Before starting momentum, take a break. Walk around. Then sit down and answer these from memory:

- What library formats the Excel output?
- How do you split a list into chunks of 100 in Python?
- What's the formula for number of shares to buy?

---

#### Session 2B: Quantitative Momentum Strategy — Concepts + Start (1:38:44 → ~2:15:00) — ~36 min watch

**What you're learning:**

- What momentum investing is
- One-year price returns vs. more granular momentum (HQM)
- Pulling return data from the API

**The analogy:** Momentum investing is like betting on a horse that's *already winning the race*. The idea — backed by real academic research — is that stocks going up tend to keep going up (for a while). You're not predicting; you're surfing a wave that already started.

**Why it matters (the critical "why"):** Momentum is one of the few market anomalies that has survived academic scrutiny for decades. Jegadeesh & Titman (1993) documented it. It shouldn't work in efficient markets, but it does — probably because humans are slow to update beliefs and tend to herd.

**Common mistake:** Simple momentum (just one-year return) is noisy. A stock could have gone up 200% in January and done nothing since. That's why the course introduces *High Quality Momentum (HQM)* — using multiple time windows (1-month, 3-month, 6-month, 1-year) to find stocks with *consistent* momentum, not just one big spike.

**Do this:**

1. Watch at 1x for the conceptual explanation (this is new territory)
2. Start coding along when he pulls return data
3. **Stop here for the day** — let the concepts incubate overnight

**Before bed, explain to yourself:** What's the difference between simple momentum and high-quality momentum? Why does the multi-period approach matter?

---

### DAY 3 — Finish Momentum + Value Strategy (Part 1)

**Time:** ~2.5 hours | **Video:** ~2:15:00 → ~3:30:00

#### Morning recall (10 min, no notes)

Write down everything you remember about:

- Equal-weight fund: how it works, key code pattern
- Momentum: what HQM is and why multi-period matters

Then check your notes. **The gaps you find are what to pay extra attention to today.**

---

#### Session 3A: Finish Quantitative Momentum (→ 2:54:02) — ~39 min watch

**What you're learning:**

- Calculating percentile scores for each return period
- Combining percentile scores into an HQM score
- Selecting top 50 momentum stocks
- Outputting to Excel

**The pattern:** `raw metric → percentile rank → composite score → sort → select top N`. This is a generalizable ranking framework you can apply to any multi-factor strategy.

**Do this:**

1. Code along at 1.25x
2. After finishing, experiment: What happens if you weight recent momentum (1-month) more heavily? Change the composite score formula and see how the stock picks change.
3. **Rebuild from scratch** — your second project rebuilt from memory

---

#### Session 3B: Quantitative Value Strategy — Concepts + Start (2:54:02 → ~3:30:00) — ~36 min watch

**What you're learning:**

- What value investing is
- Key value metrics: P/E ratio, P/B ratio, P/S ratio, EV/EBITDA, EV/GP
- Why using multiple value metrics beats using just one

**The analogy:** Value investing is like shopping at a thrift store — you're looking for a designer jacket priced like a t-shirt. The "price" is the stock price; the "designer jacket" is the company's actual earnings, book value, or sales. If the price is low relative to what the company actually produces, it might be undervalued.

**First principles:**

- P/E ratio = Price / Earnings. If a company earns $10/share and trades at $100, P/E = 10. If it trades at $50, P/E = 5 — "cheaper."
- But P/E alone is misleading (negative earnings, cyclical industries, etc.). That's why you use 5 metrics together — same logic as HQM using multiple time periods.

**The gap most people miss:** Value and momentum are *philosophically opposite*. Momentum says "buy winners." Value says "buy losers (that are underpriced)." The interesting thing is both work historically, and they work *even better* when combined (because they tend to zig when the other zags).

**Do this:**

1. Watch at 1x for concepts, 1.25x for code
2. Map the code structure mentally to the momentum project — notice how similar the architecture is
3. **Stop here for the day**

---

### DAY 4 — Finish Value Strategy + Integration & Review

**Time:** ~2.5 hours | **Video:** ~3:30:00 → 4:33:02

#### Morning recall (10 min)

From memory, sketch the code architecture shared by all three projects:

1. _____ (get stock list)
2. _____ (batch API calls)
3. _____ (parse into DataFrame)
4. _____ (calculate metrics/scores)
5. _____ (select top N)
6. _____ (calculate position sizes)
7. _____ (output to Excel)

---

#### Session 4A: Finish Quantitative Value (→ 4:33:02) — ~63 min watch

**What you're learning:**

- Calculating percentile scores for 5 value metrics
- Handling missing data (NaN values)
- Building a composite "RV Score" (Robust Value)
- Final Excel output

**Do this:**

1. Code along at 1.25x
2. Pay special attention to NaN handling — real financial data is messy, and this is where production code breaks
3. After finishing, **rebuild from scratch** — third project rebuilt from memory

---

#### Session 4B: Integration Review (no video — your own work) — ~45 min

This is where the real learning locks in.

**Exercise 1 — Compare the three strategies (15 min):**
Create a table from memory:

| | Equal-Weight | Momentum | Value |
| --- | --- | --- | --- |
| Philosophy | ? | ? | ? |
| Key metrics | ? | ? | ? |
| When it works best | ? | ? | ? |
| Main risk | ? | ? | ? |

**Exercise 2 — Teach it back (15 min):**
Explain to an imaginary friend (out loud, seriously) how you'd build a quantitative momentum strategy. Cover: what data you need, how you score stocks, why you use multiple time periods, and how you decide how many shares to buy.

**Exercise 3 — Challenge it (15 min):**
Write down 3 things this course *doesn't* address that you'd need for a real trading system:

- (Hint: transaction costs, slippage, backtesting, risk management, rebalancing frequency, tax implications, survivorship bias in the S&P 500 list)

---

## Spaced Repetition Schedule (After Day 4)

| When | What to review |
| ------ | --------------- |
| Day 5 (next day) | Rebuild ONE project from scratch (pick your weakest) |
| Day 8 (3 days later) | Explain all 3 strategies out loud from memory, no notes |
| Day 15 (1 week later) | Modify one project: add a new metric, change the scoring, or combine momentum + value |
| Day 30 (2 weeks later) | Build a NEW strategy from scratch using the same framework (e.g., dividend yield + growth) |

---

## Quick Reference: Key Python Patterns from This Course

| Pattern | What it does | Reuse it for |
| --------- | ------------- | ------------- |
| `list(chunks(tickers, 100))` | Split list into batches | Any bulk API work |
| `pd.DataFrame(columns=[...])` | Initialize structured data | Any tabular data pipeline |
| `scipy.stats.percentileofscore()` | Rank a value within a distribution | Any multi-factor scoring |
| `xlsxwriter` formatting | Styled Excel output | Client-facing reports |
| `requests.get(url).json()` | API call + parse | Any REST API integration |

---

## Common Mistakes & Misunderstandings

1. **"This will make me money"** — The course uses scrambled test data. Real algo trading requires backtesting, risk management, and dealing with execution costs. This course teaches the *framework*, not a ready-to-deploy system.
2. **Overfitting to percentiles** — Ranking by percentile works for stock selection, but the composite scores (HQM, RV) weight all metrics equally. In practice, you'd want to test different weightings.
3. **Ignoring the API changes** — IEX Cloud's free tier has changed since 2020. If the API calls don't work, check the [IEX Cloud docs](https://iexcloud.io/docs/) or substitute with `yfinance` or another free data source.
4. **Copy-pasting instead of typing** — The course has a GitHub repo. Using it as a reference is fine; using it as a shortcut kills the learning.

---

*Generated for accelerated learning. Watch speed recommendations assume advanced Python familiarity.*
