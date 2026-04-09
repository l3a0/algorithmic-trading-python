# Algorithmic Trading with Python — Detailed Notes

**Course:** [freeCodeCamp - Algorithmic Trading Using Python](https://www.youtube.com/watch?v=xfzGZB4HhEE)
**Companion to:** algo-trading-python-study-plan.md

---

## SECTION 1: Algorithmic Trading Fundamentals & API Basics (0:00–17:20)

### 1.1 What Is Algorithmic Trading?

**Definition:** Using computer programs to execute trades based on predefined rules, at speeds and frequencies impossible for humans.

**First principles breakdown:** Every trade has three decisions — *what* to buy/sell, *when* to do it, and *how much*. Algorithmic trading automates all three by encoding them as rules that a computer executes.

**Types of algorithmic trading (from simple to complex):**

| Type | What it does | Example |
| ---- | ------------ | ------- |
| Rule-based execution | Automates manual decisions | "Buy 100 shares of AAPL if price < $150" |
| Quantitative screening | Ranks stocks by metrics | This course's approach |
| Statistical arbitrage | Exploits price relationships | Pairs trading (Coca-Cola vs. Pepsi) |
| High-frequency trading (HFT) | Sub-millisecond execution | Market making, latency arbitrage |
| Machine learning | Learns patterns from data | Neural nets predicting price movement |

**Where this course fits:** Quantitative screening. You're not building an HFT system or a neural network. You're automating the process of "which stocks should I buy, and how many shares of each?"

**The mental image:** Think of a funnel. 500 stocks go in at the top. Your algorithm filters and scores them. 50 stocks come out at the bottom with exact share counts. That's what you're building three times, with three different filters.

### 1.2 API Basics

**What is a REST API?** A way for your Python code to ask a server for data over the internet. You send an HTTP request (like typing a URL), and get back structured data (JSON).

**Anatomy of a stock API call:**

```text
REQUEST:  GET https://api.example.com/stock/AAPL/quote?token=YOUR_KEY
RESPONSE: {
    "symbol": "AAPL",
    "latestPrice": 178.52,
    "marketCap": 2780000000000,
    "peRatio": 29.5,
    "change": 1.23,
    "changePercent": 0.00693
}
```

**Key Python pattern:**

```python
import requests

url = "https://api.example.com/stock/AAPL/quote"
params = {"token": "YOUR_API_KEY"}
response = requests.get(url, params=params)
data = response.json()  # dict

price = data["latestPrice"]  # 178.52
```

**Critical note (2026):** The course uses IEX Cloud, which shut down August 31, 2024. You need an alternative:

| Alternative | Free tier | Best for |
| ------------ | --------- | -------- |
| **Alpha Vantage** | 25 calls/day | Direct IEX migration (has migration guide) |
| **Finnhub** | 60 calls/min | Real-time quotes |
| **Financial Modeling Prep** | Limited | Deep fundamentals (P/E, P/B, etc.) |
| **EODHD** | Basic EOD data | Broadest coverage, good batch support |
| **yfinance** | Unlimited (scraping) | Quick prototyping only — unreliable for production |

### 1.3 Environment Setup

**Required libraries:**

```bash
pip install pandas numpy requests xlsxwriter scipy
```

**What each does:**

- `pandas` — DataFrames (your tabular data structure for all 3 projects)
- `numpy` — Numerical operations, NaN handling
- `requests` — HTTP API calls
- `xlsxwriter` — Formatted Excel output
- `scipy` — `percentileofscore()` for ranking stocks

**Project structure:**

```text
algorithmic-trading-python/
  starter_files/        # Templates with TODOs
  finished_files/       # Completed solutions
  sp_500_stocks.csv     # S&P 500 ticker list
  secrets.py            # API key (gitignored)
```

---

## SECTION 2: Building an Equal-Weight S&P 500 Index Fund (17:20–1:38:44)

### 2.1 The Concept: Equal-Weight vs. Market-Cap Weight

**Market-cap weighted (how most index funds work):**

- Each stock's weight = its market cap / total market cap of all stocks
- Apple (~$3T) gets ~7% of the index; a small company gets <0.1%
- Dominated by "Magnificent Seven" (Apple, Microsoft, Nvidia, Amazon, Alphabet, Meta, Tesla)
- As of late 2024, top 10 stocks = 35% of the entire S&P 500

**Equal-weight (what you're building):**

- Each stock's weight = 1/500 = 0.2%
- Apple gets the same weight as the smallest S&P 500 company
- Must rebalance quarterly (prices drift, weights shift)

**Historical performance comparison:**

| Metric | Market-cap weighted | Equal-weight |
| -------- | ------------------- | -------------- |
| Avg annual return (1990–2023) | ~10.5% | ~11.1% |
| Volatility | Lower | ~20-30% higher |
| Turnover | ~10-15%/year | ~50-150%/year |
| Expense ratio (ETF) | 0.03% (VOO) | 0.20% (RSP) |
| Recent performance (2022-2024) | Outperformed (Mag 7 rally) | Underperformed by ~13% |

**Why equal-weight sometimes wins:** It gives more weight to smaller companies (the "size effect"), and it systematically sells winners and buys losers at each rebalance (a contrarian/mean-reversion tilt).

**Why it sometimes loses:** When a few mega-cap stocks drive the market (like the Mag 7 era), equal-weight misses the concentration benefit.

**The key insight:** Equal-weight is not "better" — it's a *different bet*. Market-cap weight bets on big winners staying big. Equal-weight bets on the average stock catching up.

### 2.2 The Code Architecture

This is the template for ALL three projects:

```text
Step 1: Load stock list (sp_500_stocks.csv)
Step 2: Pull data from API (batch calls)
Step 3: Store in pandas DataFrame
Step 4: Calculate position sizes
Step 5: Output to formatted Excel
```

**Step 1 — Load tickers:**

```python
import pandas as pd

stocks = pd.read_csv('sp_500_stocks.csv')
# DataFrame with column 'Ticker': ['AAPL', 'MSFT', 'GOOGL', ...]
```

**Step 2 — The batch API call pattern (the most important pattern in this course):**

*Problem:* 500 individual API calls = slow, rate-limited, wasteful.
*Solution:* Batch tickers into groups of 100 and make 5 calls instead of 500.

```python
def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]

symbol_groups = list(chunks(stocks['Ticker'], 100))
# 5 groups of ~100 tickers each

for group in symbol_groups:
    symbol_string = ','.join(group)
    batch_url = f"https://api.example.com/stock/market/batch?symbols={symbol_string}&types=quote"
    data = requests.get(batch_url, params={"token": API_KEY}).json()
    # Parse response and append to DataFrame
```

**Why this matters beyond this course:** Chunking + batch is a universal data engineering pattern. Any time you need to process N items through a rate-limited service, you chunk them.

**Step 3 — Build the DataFrame:**

```python
columns = ['Ticker', 'Stock Price', 'Market Capitalization', 'Number of Shares to Buy']
df = pd.DataFrame(columns=columns)

for symbol in data:
    df = df.append(
        pd.Series([
            symbol,
            data[symbol]['quote']['latestPrice'],
            data[symbol]['quote']['marketCap'],
            'N/A'  # Calculate later
        ], index=columns),
        ignore_index=True
    )
```

*Note:* `df.append()` is deprecated in newer pandas. Use `pd.concat()` instead:

```python
new_row = pd.DataFrame([{
    'Ticker': symbol,
    'Stock Price': data[symbol]['quote']['latestPrice'],
    'Market Capitalization': data[symbol]['quote']['marketCap'],
    'Number of Shares to Buy': 'N/A'
}])
df = pd.concat([df, new_row], ignore_index=True)
```

**Step 4 — Calculate position sizes:**

```python
portfolio_size = float(input("Enter your portfolio value: "))  # e.g., 1000000
position_size = portfolio_size / len(df)  # Equal weight = total / number of stocks

for i in range(len(df)):
    df.loc[i, 'Number of Shares to Buy'] = math.floor(
        position_size / df.loc[i, 'Stock Price']
    )
```

**The formula:** `shares = floor(position_size / stock_price)`

Using `floor()` because you can't buy fractional shares in this model (though many brokers now support fractional shares).

**Step 5 — Excel output with xlsxwriter:**

```python
writer = pd.ExcelWriter('recommended_trades.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Recommended Trades', index=False)

# Format columns
workbook = writer.book
worksheet = writer.sheets['Recommended Trades']

background_color = '#0a0a23'
font_color = '#ffffff'

string_format = workbook.add_format({
    'font_color': font_color,
    'bg_color': background_color,
    'border': 1
})

dollar_format = workbook.add_format({
    'num_format': '$#,##0.00',
    'font_color': font_color,
    'bg_color': background_color,
    'border': 1
})

# Apply formats to columns
worksheet.set_column('A:A', 18, string_format)   # Ticker
worksheet.set_column('B:B', 18, dollar_format)    # Price
worksheet.set_column('C:C', 18, dollar_format)    # Market Cap
worksheet.set_column('D:D', 18, string_format)    # Shares

writer.save()
```

### 2.3 What Most People Miss

- **Rebalancing cost:** An equal-weight portfolio of 500 stocks has 5-10x more turnover than market-cap weight. Transaction costs eat into the theoretical advantage.
- **`df.append()` deprecation:** The course uses `df.append()` which was removed in pandas 2.0. Replace with `pd.concat()`.
- **Fractional shares:** The course uses `math.floor()`, meaning leftover cash sits uninvested. Modern brokers (Robinhood, Schwab) support fractional shares, which would give more precise equal weighting.

---

## SECTION 3: Building a Quantitative Momentum Investing Strategy (1:38:44–2:54:02)

### 3.1 What Is Momentum Investing?

**Core idea:** Stocks that have gone up recently tend to keep going up. Stocks that have gone down tend to keep going down. Buy the winners.

**Academic foundation — Jegadeesh & Titman (1993):**

- Tested all combinations of 3, 6, 9, and 12-month formation periods
- Found that buying past winners and shorting past losers generated ~1.5% monthly excess returns
- Works across virtually all developed and emerging markets
- One of the most robust anomalies in finance — survived 30+ years of scrutiny
- [Original paper (PDF)](https://www.bauer.uh.edu/rsusmel/phd/jegadeesh-titman93.pdf)

**Why does momentum work? Three leading explanations:**

1. **Behavioral:** Investors underreact to good news initially, then herd into winning stocks (overreaction), creating a trend
2. **Risk-based:** Momentum stocks are riskier (they crash hard in reversals), so the higher return is compensation for risk
3. **Information diffusion:** Good news spreads slowly, especially for smaller or less-covered companies

**The analogy:** A snowball rolling downhill. It picks up speed not because of magic, but because each layer of snow (new buyers) adds mass. Eventually it hits a wall (reversal), but while it's rolling, the physics favor continuation.

### 3.2 Simple Momentum vs. High-Quality Momentum (HQM)

**Simple momentum:** Rank stocks by one-year return. Buy the top 50.

*Problem:* A stock that went up 300% in January and fell 50% since would still show a high one-year return. It had a spike, not a trend.

**High-Quality Momentum (HQM):** Rank stocks by *consistency* of momentum across multiple time windows.

| Time window | What it captures |
| ------------- | ----------------- |
| 1-month return | Very recent trend |
| 3-month return | Short-term trend |
| 6-month return | Medium-term trend |
| 1-year return | Long-term trend |

**HQM score = average of percentile ranks across all four windows.**

A stock ranked 90th percentile in ALL four windows has *consistent, high-quality* momentum. A stock ranked 99th in 1-year but 20th in 1-month has *fading* momentum.

### 3.3 Percentile Scoring — The Core Algorithm

**Why percentiles instead of raw returns?**

- Raw returns aren't comparable across time windows (a 5% monthly return is great; a 5% annual return is terrible)
- Percentiles normalize everything to 0–100
- Makes combining metrics trivial (just average the percentiles)

**How `scipy.stats.percentileofscore()` works:**

```python
from scipy.stats import percentileofscore

all_returns = [5, 8, 12, 15, 20, 3, 7, 18, -2, 25]
percentileofscore(all_returns, 12)
# Returns ~55.0 — 12 is better than ~55% of stocks
```

**Building the HQM score:**

```python
from scipy.stats import percentileofscore as score

time_periods = ['One-Year', 'Six-Month', 'Three-Month', 'One-Month']

for row in df.index:
    for period in time_periods:
        # Get this stock's return for this period
        change_col = f'{period} Price Return'
        pctl_col = f'{period} Return Percentile'

        df.loc[row, pctl_col] = score(
            df[change_col],           # All stocks' returns
            df.loc[row, change_col]   # This stock's return
        )

    # HQM Score = mean of all four percentiles
    df.loc[row, 'HQM Score'] = df.loc[row, [
        'One-Year Return Percentile',
        'Six-Month Return Percentile',
        'Three-Month Return Percentile',
        'One-Month Return Percentile'
    ]].mean()
```

**Select top 50 by HQM score:**

```python
df.sort_values('HQM Score', ascending=False, inplace=True)
df = df[:50]
df.reset_index(drop=True, inplace=True)
```

### 3.4 Momentum Crashes — The Critical Risk

Momentum strategies can crash *hard* during market reversals:

| Event | Momentum drawdown |
| ------- | ------------------ |
| 2001-2002 (dot-com bust) | ~28% |
| 2009 (financial crisis recovery) | ~54% |
| 2020 (COVID recovery) | Significant (value snapped back) |

**Why crashes happen:** When the market reverses sharply, yesterday's losers become today's winners, and yesterday's winners fall. Momentum is *long* the winners and *short* the losers — exactly the wrong position.

**Mitigations (not covered in the course, but crucial):**

- **Volatility scaling:** Reduce momentum exposure when market volatility is high
- **Combine with value:** Value and momentum crash at different times
- **Liquidity filter:** Avoid illiquid small-caps that amplify crashes
- **Stop-losses:** Simple but effective floor on individual positions

### 3.5 What Most People Miss

- **Momentum is academically robust but practically dangerous** without risk management. The course doesn't cover stop-losses, position sizing by risk, or drawdown limits.
- **Tax drag:** Momentum strategies have high turnover (buying new winners, selling old ones quarterly). In taxable accounts, this generates short-term capital gains taxed at ordinary income rates.
- **The "skip-a-month" effect:** Many practitioners skip the most recent month when calculating momentum (use months 2-12 instead of 1-12) because the very last month tends to show short-term reversal.

---

## SECTION 4: Building a Quantitative Value Investing Strategy (2:54:02–4:33:02)

### 4.1 What Is Value Investing?

**Core idea:** Some stocks are priced below their intrinsic worth. If you can identify them, you buy cheap and wait for the market to recognize the true value.

**First principles:** A company has real assets, real earnings, and real cash flow. The stock price is what people *think* the company is worth. If the price is low relative to what the company actually produces, it might be a bargain.

**The analogy:** Imagine a house appraised at $500K that's listed for $300K because the owners need to sell quickly. A value investor would say: "The fundamentals (location, square footage, condition) are worth $500K. The current price is a temporary discount. I'll buy."

**Academic foundation — Fama & French (1993):**

- Discovered that stocks with high book-to-market ratios (value stocks) outperform stocks with low book-to-market ratios (growth stocks)
- This "value premium" has been ~4-5% annually over decades
- Led to the famous three-factor model (market risk + size + value)
- Later expanded to five factors adding profitability and investment patterns

### 4.2 The Five Value Metrics

#### 1. P/E Ratio (Price to Earnings)

```text
P/E = Stock Price / Earnings Per Share
```

- Measures: How much you pay per dollar of profit
- Low P/E = "cheap" relative to earnings
- Example: Stock at $50 earning $5/share → P/E = 10
- **Limitation:** Meaningless for unprofitable companies (negative earnings). Cyclical companies may appear cheap at peak earnings.
- **Best for:** Stable, profitable companies

#### 2. P/B Ratio (Price to Book Value)

```text
P/B = Market Cap / (Total Assets - Total Liabilities)
```

- Measures: How much you pay relative to the company's net asset value
- Low P/B = stock trades near or below liquidation value
- P/B < 1 means the market values the company at less than its assets
- **Limitation:** Less meaningful for asset-light tech companies (most value is in IP, not physical assets)
- **Best for:** Banks, manufacturing, real estate

#### 3. P/S Ratio (Price to Sales)

```text
P/S = Market Cap / Annual Revenue
```

- Measures: How much you pay per dollar of revenue
- Works even for unprofitable companies (everyone has revenue)
- More stable than P/E (revenue is harder to manipulate than earnings)
- **Limitation:** Ignores whether the company is actually profitable
- **Best for:** High-growth or unprofitable companies where P/E doesn't work

#### 4. EV/EBITDA (Enterprise Value to EBITDA)

```text
EV = Market Cap + Total Debt - Cash
EBITDA = Earnings Before Interest, Taxes, Depreciation, Amortization
EV/EBITDA = EV / EBITDA
```

- Measures: Total company valuation relative to operating cash generation
- **Why EV instead of market cap?** EV accounts for debt. A company with $100M market cap and $900M debt is effectively a $1B acquisition — EV captures that.
- **Why EBITDA instead of earnings?** Removes the effect of capital structure (debt vs. equity) and accounting choices (depreciation methods).
- Considered the "gold standard" single metric by M&A professionals
- Typical benchmark: EV/EBITDA of ~6x is considered conservative value
- **Limitation:** Can be misleading for capital-intensive businesses where depreciation is a real cost

#### 5. EV/GP (Enterprise Value to Gross Profit)

```text
Gross Profit = Revenue - Cost of Goods Sold
EV/GP = EV / Gross Profit
```

- Measures: Valuation relative to production efficiency
- More conservative than EV/EBITDA (gross profit is higher up the income statement)
- Captures the company's core economic engine before operating expenses
- **Limitation:** Less commonly used, fewer benchmarks available
- **Best for:** Comparing companies with different operating expense structures

### 4.3 Why Composite Scores Beat Any Single Metric

**Research finding (O'Shaughnessy, "What Works on Wall Street"):**
The "Value Composite Two" (VC2) — averaging percentile ranks across multiple value metrics — outperformed any single value metric **82% of the time** across all 10-year rolling periods from 1964-2009.

**Why no single metric is enough:**

| Metric | Fails when... |
| -------- | ------------- |
| P/E | Earnings are negative, cyclical, or manipulated |
| P/B | Company is asset-light (tech, services) |
| P/S | Company has revenue but burns cash |
| EV/EBITDA | Heavy capex makes EBITDA misleading |
| EV/GP | Operating costs are the real problem |

**The pattern:** Each metric has blind spots. A stock that looks cheap on ALL five metrics is almost certainly genuinely cheap. A stock that looks cheap on just one might have a specific problem that one metric doesn't see.

### 4.4 The Robust Value (RV) Score — Code Architecture

Same pattern as HQM, just with value metrics instead of return periods:

```python
value_metrics = ['P/E Ratio', 'P/B Ratio', 'P/S Ratio', 'EV/EBITDA', 'EV/GP']

for row in df.index:
    for metric in value_metrics:
        pctl_col = f'{metric} Percentile'
        df.loc[row, pctl_col] = score(df[metric], df.loc[row, metric])

    df.loc[row, 'RV Score'] = df.loc[row, [
        'P/E Ratio Percentile',
        'P/B Ratio Percentile',
        'P/S Ratio Percentile',
        'EV/EBITDA Percentile',
        'EV/GP Percentile'
    ]].mean()
```

**Critical difference from momentum:** For value, LOWER is cheaper. A stock at the 5th percentile of P/E is CHEAP (low price relative to earnings). So you select stocks with the **lowest** RV score, not the highest.

```python
df.sort_values('RV Score', ascending=True, inplace=True)  # ascending=True for value!
df = df[:50]
```

### 4.5 Handling NaN Values — The Real-World Challenge

Financial data is messy. Companies may have:

- Negative earnings (P/E is meaningless)
- No EBITDA reported
- Missing book value data

**The course's approach:**

```python
# Replace NaN with the column mean (simple imputation)
for column in ['P/E Ratio', 'P/B Ratio', 'P/S Ratio', 'EV/EBITDA', 'EV/GP']:
    df[column].fillna(df[column].mean(), inplace=True)
```

**Why this works for screening:** You're ranking 500 stocks. Imputing the mean for missing values puts those stocks in the middle of the pack — they won't be selected as top value picks or excluded entirely. It's a reasonable default.

**Better approaches (not in the course):**

- **Exclude stocks with >2 missing metrics** — if most of their data is missing, the composite score is unreliable
- **Use `pd.Series.rank(pct=True, na_option='keep')`** — rank within available data, keep NaN as NaN
- **Winsorize outliers** — cap extreme values at the 1st/99th percentile before ranking

### 4.6 Value vs. Momentum — The Philosophical Tension

| | Momentum | Value |
| --- | --------- | ------- |
| Philosophy | "Buy winners" | "Buy losers (that are underpriced)" |
| Behavioral basis | Herding, underreaction | Overreaction, panic selling |
| Works because | Trends persist | Mean reversion |
| Crashes when | Market reverses sharply | Value keeps getting cheaper (value trap) |
| Typical holding period | 3-12 months | 1-5 years |
| Turnover | High | Moderate |

**The key insight most people miss:** These two strategies are *negatively correlated*. When momentum crashes (market reversal), value tends to do well (cheap stocks bounce back). When value underperforms (growth stocks dominate), momentum captures the growth trend.

**Implication:** Combining momentum and value in a single portfolio provides better risk-adjusted returns than either alone. This is the foundation of modern multi-factor investing.

---

## SECTION 5: Cross-Cutting Patterns & Technical Reference

### 5.1 The Universal Code Architecture

All three projects follow this exact skeleton:

```python
# 1. SETUP
import pandas as pd, numpy as np, requests, math
from scipy.stats import percentileofscore as score

stocks = pd.read_csv('sp_500_stocks.csv')
API_KEY = 'your_key'

# 2. BATCH DATA RETRIEVAL
def chunks(lst, n):
    for i in range(0, len(lst), n):
        yield lst[i:i+n]

columns = [...]  # Project-specific columns
df = pd.DataFrame(columns=columns)

for group in chunks(stocks['Ticker'], 100):
    symbols = ','.join(group)
    url = f"api_url/batch?symbols={symbols}&types=quote,stats"
    data = requests.get(url, params={'token': API_KEY}).json()

    for symbol in group:
        # Parse and append to df

# 3. SCORING (Momentum or Value specific)
for row in df.index:
    for metric in metrics:
        df.loc[row, f'{metric} Percentile'] = score(df[metric], df.loc[row, metric])
    df.loc[row, 'Composite Score'] = df.loc[row, percentile_columns].mean()

# 4. SELECTION
df.sort_values('Composite Score', ascending=..., inplace=True)
df = df[:50]

# 5. POSITION SIZING
portfolio_size = float(input("Enter portfolio value: "))
position_size = portfolio_size / len(df)
for i in range(len(df)):
    df.loc[i, 'Shares to Buy'] = math.floor(position_size / df.loc[i, 'Price'])

# 6. EXCEL OUTPUT
writer = pd.ExcelWriter('output.xlsx', engine='xlsxwriter')
df.to_excel(writer, sheet_name='Picks', index=False)
# ... formatting ...
writer.save()
```

### 5.2 Key Python Gotchas

| Issue | Problem | Solution |
| ------- | --------- | ---------- |
| `df.append()` deprecated | Removed in pandas 2.0 | Use `pd.concat([df, new_row])` |
| `percentileofscore` with NaN | Raises error or wrong result | Filter NaN before calling |
| `writer.save()` deprecated | Use context manager | `with pd.ExcelWriter(...) as writer:` |
| Integer division for shares | `//` rounds toward negative infinity | Use `math.floor(x)` for positive numbers |
| API rate limiting | Get HTTP 429 errors | Add `time.sleep(0.5)` between batch calls |

### 5.3 xlsxwriter Format Cheat Sheet

```python
# Number formats
'$#,##0.00'     # Currency: $1,234.56
'#,##0'         # Integer with commas: 1,234
'0.00%'         # Percentage: 12.34%
'0.0'           # One decimal: 12.3

# Colors (freeCodeCamp dark theme)
background = '#0a0a23'
font_color = '#ffffff'

# Creating formats
fmt = workbook.add_format({
    'font_color': font_color,
    'bg_color': background,
    'border': 1,
    'num_format': '$#,##0.00'
})

# Applying to columns
worksheet.set_column('A:A', 18, string_fmt)   # Ticker
worksheet.set_column('B:B', 18, dollar_fmt)    # Price
worksheet.set_column('C:C', 18, percent_fmt)   # Returns
worksheet.write_row(0, 0, headers, header_fmt) # Header row
```

---

## SECTION 6: What This Course Doesn't Cover (Critical Gaps)

Understanding what's *missing* is as important as understanding what's taught:

**1. Backtesting**
The course builds a stock screener, not a backtested strategy. You don't know if these picks *would have* made money historically. Libraries like `backtrader`, `zipline`, or `vectorbt` let you test strategies on historical data.

**2. Transaction costs & slippage**
Every trade costs money (commissions, bid-ask spread, market impact). The course assumes frictionless trading. Real-world impact: 0.1-0.5% per trade, which compounds significantly with high turnover.

**3. Risk management**
No stop-losses, no maximum position sizes, no portfolio volatility targeting. A real system needs rules like "never put more than 5% in one stock" or "reduce exposure when portfolio drawdown exceeds 10%."

**4. Rebalancing frequency**
The course is a one-time snapshot. Real strategies rebalance monthly or quarterly. Each rebalance generates new picks, new trades, and new costs.

**5. Survivorship bias**
The S&P 500 list *today* only includes companies that survived to today. Companies that went bankrupt were removed. Backtesting on today's list inflates historical returns. A proper backtest uses the historical S&P 500 composition at each point in time.

**6. Tax implications**
Short-term capital gains (holding < 1 year) are taxed at ordinary income rates (up to 37% in the US). Momentum's high turnover generates mostly short-term gains. Value's longer holding periods can qualify for long-term rates (15-20%).

**7. Execution**
The course generates a spreadsheet. Actually placing 50 trades requires a brokerage API (Alpaca, Interactive Brokers, TD Ameritrade). Order types matter: market orders get slippage; limit orders might not fill.

---

*These notes are designed to complement the video course, not replace it. Watch the video for the hands-on coding walkthrough; use these notes for the conceptual depth and context the video moves through quickly.*
