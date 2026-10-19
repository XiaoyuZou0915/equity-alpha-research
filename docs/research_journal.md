# Equity Alpha Research — Research Journal

## 2026-10-12 | Week 1 Day 1 — Data Exploration

### Objective

Establish a reproducible local research environment, acquire a small daily OHLCV sample, inspect its structure, and identify initial data quality and corporate-action considerations.

### Work Completed

Verified the macOS project-specific Python virtual environment and VS Code/Jupyter kernel.

Acquired daily Yahoo Finance sample data using Google Colab after encountering a local YFRateLimitError.

Exported the data as CSV and imported it into the local research project.

Inspected a Pandas DataFrame with MultiIndex columns.

Conducted preliminary date, missing-value, price, and volume checks.

Compared XOM's Close and Adj Close.

### Dataset

Tickers: AAPL, MSFT, JPM, XOM, WMT

Frequency: Daily

Date range: 2020-01-02 to 2024-12-31

Shape: 1,258 rows × 40 columns

Source: Yahoo Finance via yfinance, downloaded in Google Colab

Purpose: Data exploration and pipeline testing only

### Preliminary Findings

The date index is ordered and contains no duplicates.

No missing values were found in the sample.

No non-positive closing prices or negative/zero trading volumes were found.

XOM's Adj Close / Close adjustment ratio differs across historical dates, highlighting the importance of corporate-action-aware return calculations.

Data access is environment-dependent: the download succeeded in Colab but was rate-limited locally.

### Limitations

The five selected tickers form a convenience sample, not a historically reconstructed point-in-time universe.

Survivorship bias and delisted-stock coverage are unresolved.

Corporate-action adjustments have not been independently audited.

Trading-calendar completeness and vendor data accuracy have not yet been independently verified.

Results from this sample must not be interpreted as evidence of alpha.

### Open Questions

Which data source can support a historical, delisted-inclusive US equity universe?

Which price and return conventions should the main study use?

How should feature availability, signal generation, execution, and label maturity be recorded?

How should missing prices, trading suspensions, and delistings be handled?

### Next Actions

Draft the research protocol with explicit signal, decision, execution, and exit timestamps.

Compare candidate historical equity data sources.

Define a preliminary data dictionary and additional data-quality tests.

---

## 2026-10-17 | Week 1 Day 3 — Corporate Actions & Point-in-Time Data

**Status:** Completed  
**Date:** 2026-10-17

### 1. Corporate Actions Audit

Examined dividend and stock split events in the five-stock exploratory dataset (AAPL, MSFT, JPM, XOM, WMT), covering 2020–2024.

Findings:

- Each security had 20 recorded dividend events.
- AAPL had a 4-for-1 stock split on 2020-08-31.
- WMT had a 3-for-1 stock split on 2024-02-26.
- The dataset contained 100 dividend events and 2 split events in total.

### 2. AAPL Stock Split Experiment

Compared Close and Adj Close returns around the 2020-08-31 split.

Findings:

- AAPL Close Return on the split date was approximately +3.39%, not -75%.
- Close Return and Adj Return were approximately equal.
- Yahoo's historical Close series was already split-adjusted.
- Applying the split ratio again could lead to double adjustment.

### 3. XOM Dividend Experiment

Examined XOM's $0.87-per-share dividend event on 2020-02-10.

Results:

| Return Definition | Return |
|---|---:|
| Close Return | -2.456% |
| Adjusted Return | -1.056% |
| Simplified Dividend-Inclusive Return | -1.041% |

The adjusted return exceeded the close return by approximately 1.40 percentage points.

This demonstrated the importance of distinguishing price returns from dividend-inclusive returns.

### 4. Point-in-Time Data Audit

Investigated the implications of historical corporate-action adjustments.

Key findings:

- A historical observation date is not necessarily the data availability date.
- Future splits may retroactively change historical adjusted price levels.
- A uniform multiplicative adjustment to both prices does not change a price-ratio momentum feature.
- Absolute historical price thresholds can be misclassified if retrospectively split-adjusted prices are mistaken for original quoted prices.
- Feature construction must distinguish historical information availability from later vendor revisions.

### 5. Data Dictionary

Created `docs/data_dictionary_v0.md`.

The document defines preliminary OHLCV, dividend, split, return and point-in-time conventions, together with known data limitations.

### 6. Production Data Source Audit

**CRSP / WRDS**

- No independent WRDS account is currently available.
- Cornell Alumni access does not establish CRSP authorization.
- Formal CRSP access remains unconfirmed.

**Sharadar SEP**

- Identified as a potential alternative source for historical US equities.
- Published documentation describes adjusted and unadjusted price fields, historical delisted securities and corporate-action data.
- Point-in-time completeness, delisting-return coverage, subscription costs and licensing terms remain unverified.

**Yahoo Finance**

- Retained for exploratory analysis only.
- Not approved as the production data source for final research inference.

### 7. Open Questions

- Which production data provider is economically and technically feasible?
- Can the provider reliably reconstruct historical security eligibility?
- How should delisting returns and missing execution prices be handled?
- Can corporate-action adjustments be reconstructed using information available at each historical cutoff?
- What return convention should be used consistently for model targets and portfolio evaluation?

### 8. Next Steps

- Continue the production data provider feasibility assessment.
- Specify the security identifier and point-in-time universe schema.
- Define data validation checks before formal signal development.

## Week 1 Day 4 — Security Master and PIT Universe Prototype

**Status:** Completed 
**Date:** 2026-10-19

### 1. Objective

Develop a synthetic Security Master and a preliminary point-in-time universe construction function.

The purpose is to validate historical security eligibility rules before integrating a production equity data provider.

### 2. Synthetic Security Master

Created `notebooks/02_pit_universe.ipynb`.

The synthetic dataset contains five securities:

| Security | Key Property |
|---|---|
| ALPHA | NYSE common stock, listed since 2010 |
| BETA | NASDAQ common stock, delisted in 2018 |
| GAMMA | NASDAQ common stock, listed in 2020 |
| DELTA | ETF |
| EPSILON | OTC common stock |

The dataset uses stable internal security IDs and simplified listing metadata.

### 3. Point-in-Time Universe Function

Implemented:

`get_pit_universe(security_master, as_of_date)`

The function filters securities based on:

- Listing date.
- Delisting date.
- Eligible exchange.
- Common-share classification.

Observed results:

| Date | Eligible Universe |
|---|---|
| 2015-06-30 | ALPHA, BETA |
| 2019-06-30 | ALPHA |
| 2021-06-30 | ALPHA, GAMMA |

Boundary checks confirmed the expected behavior around listing and delisting dates.

### 4. Automated Tests

Added tests for historical universe membership, future IPO exclusion, security classification, and listing/delisting boundaries.

Added a deliberate survivorship-bias experiment that removes delisted securities before constructing the historical universe.

**Verification status:** Confirm final assertion outputs before closing Day 4.

### 5. Key Research Findings

- Historical universe construction must depend on historical eligibility, not current survival.
- Stable security identity is distinct from the ticker symbol.
- A correct PIT filtering function cannot recover missing historical securities.
- Automated tests help detect unintended changes to eligibility logic.
- Synthetic tests do not guarantee production data is point-in-time accurate.

### 6. Current Limitations

- Historical ticker and exchange changes are not modeled.
- Price and liquidity eligibility rules are not implemented.
- Metadata availability dates are not modeled.
- Delisting returns and missing execution prices remain unresolved.
- The production equity data provider is still undecided.

### 7. Next Steps

- Design historical security-attribute records.
- Add point-in-time price and liquidity filtering.
- Extend data validation and leakage tests.
- Continue evaluating production data source feasibility.