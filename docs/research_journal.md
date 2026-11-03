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

## Week 1 Day 5 — PIT Price and Liquidity Eligibility

**Status:** Completed; Git verification pending

### 1. Objective

Extend the point-in-time universe prototype with historical price and liquidity eligibility rules.

The provisional thresholds are:

- Minimum closing price: $5.
- Minimum 60-observation median daily dollar volume: $10 million.

### 2. Synthetic Data

Created 80 synthetic business dates for five securities, producing 400 security-date observations.

Each observation contains a security ID, date, closing price, volume, and calculated dollar volume.

The dataset intentionally includes invalid post-delisting price observations for BETA.

### 3. Price Eligibility

Implemented `get_pit_price_universe()`.

The function first applies historical security eligibility, then attaches the latest available price at or before the selection cutoff.

On 2021-04-15:

- ALPHA passes the $5 minimum price.
- GAMMA fails because its price is $4.
- BETA remains excluded despite having synthetic market data.

### 4. Liquidity Eligibility

Implemented `get_pit_liquidity()`.

Daily dollar volume is calculated as:

$$
DV_{i,d}=P_{i,d}\times V_{i,d}
$$

The trailing 60-observation median dollar volume is used as the liquidity metric.

A complete 60-observation window is required.

### 5. Combined Universe

Implemented `get_pit_tradable_universe()` to combine historical security eligibility, minimum price, and minimum liquidity.

Results for 2021-04-15:

| Ticker | Close | Median Dollar Volume | Eligible |
|---|---:|---:|---|
| ALPHA | $100 | $20 million | Yes |
| GAMMA | $4 | $20 million | No |

Only ALPHA passes all provisional criteria.

### 6. Automated Tests

All seven Day 5 PIT eligibility test groups passed.

The tests covered:

- Historical universe membership.
- Combined price and liquidity screening.
- Dollar-volume calculation.
- Minimum historical observation count.
- Future price leakage.
- Exclusion of previously delisted securities.

The future price leakage test modified observations after the selection date and confirmed that the historical results remained unchanged.

### 7. Research Findings

- Price and liquidity filters must be applied jointly.
- High dollar volume does not compensate for failure to meet the minimum price requirement.
- Median dollar volume is less sensitive to isolated extreme trading days than mean dollar volume.
- Historical market data alone cannot establish security eligibility.
- Future observations must not affect historical universe construction.
- Missing or stale historical data require explicit eligibility policies.

### 8. Remaining Limitations

The current implementation uses synthetic observations and a simplified business-day calendar.

Production use requires historical security metadata, compatible price-volume adjustment conventions, explicit availability timing, trading-calendar validation, stale-price checks, and delisting-return treatment.

### 9. Next Steps

- Strengthen data quality and price-staleness checks.
- Validate complete exchange trading-day windows.
- Consider historical security attribute changes.
- Continue evaluating CRSP / Sharadar data feasibility.
- Prepare the PIT eligibility logic for future production-data integration.

---

## Week 1 Day 6 — Integration and Data Quality Review

**Status:** Completed; Git verification pending.

### 1. Objective

Evaluate the reproducibility and robustness of the Week 1 point-in-time universe prototype.

Identify outstanding engineering requirements before integrating a production equity dataset.

### 2. Notebook Reproducibility

Reviewed the importance of restarting the Python kernel and executing all notebook cells in order.

A successful isolated cell execution does not establish reproducibility because notebook variables may depend on hidden kernel state.

Added an integration summary covering security identifiers, historical universe membership, future IPO exclusion, combined eligibility, and incomplete liquidity windows.

### 3. Duplicate Record Detection

Constructed a synthetic duplicate security-date record.

Verified the duplicate-detection behavior of the liquidity function.

The preferred handling policy is to reject or quarantine conflicting records and investigate their source, rather than silently selecting one observation.

### 4. Missing and Invalid Price Tests

Tested:

- Missing closing prices.
- Zero closing prices.
- Negative closing prices.

The eligibility filter excluded these invalid values.

Added `validate_daily_prices()` to distinguish basic data-quality problems from normal eligibility failures.

### 5. Stale Price Experiment

Removed recent ALPHA observations to simulate an outdated last available price.

The existing price-selection function could still accept the historical closing price.

Added a separate price freshness check that rejects prices not satisfying the prototype's selection-date rule.

This check has not yet been integrated into the primary tradable-universe function.

### 6. Protocol Consistency Audit

Compared the Research Protocol, Data Dictionary, and synthetic Python implementation.

Identified three important gaps:

1. The 252-session minimum history requirement is not enforced.
2. Monthly universe reconstitution is not implemented.
3. Price freshness validation is not integrated into the combined eligibility pipeline.

Additional production requirements include exchange calendars, availability timestamps, historical security metadata changes, and delisting-return treatment.

### 7. Engineering Backlog

Documented the outstanding work in `docs/data_dictionary_v0.md`, Section 9.

Priorities include:

- Data integrity and leakage controls.
- Historical universe and eligibility improvements.
- Production data provider integration.

### 8. Week 1 Summary

Completed the following project milestones:

- Day 1: Environment setup and exploratory data-quality checks.
- Day 2: Research Protocol v0 and hypothesis definitions.
- Day 3: Corporate actions and point-in-time data audit.
- Day 4: Synthetic Security Master and historical universe prototype.
- Day 5: Price and liquidity eligibility prototype.
- Day 6: Integration review and abnormal-data testing.

The project now has a documented research design, exploratory market-data experiments, and a tested synthetic universe-selection prototype.

Production data access, full point-in-time reconstruction, and formal out-of-sample backtesting remain future work.

### 9. Next Steps

- Review and prioritize the engineering backlog.
- Strengthen eligibility and market-data integrity checks.
- Continue production data source feasibility assessment.
- Prepare for subsequent signal research only after core leakage controls are addressed.

---

## Week 2 Day 7 — PIT Eligibility Hardening

**Date:** 2026-11-03

**Status:** Completed; committed and pushed to origin/main.

### 1. Objectives

Strengthen the Week 1 point-in-time universe prototype by implementing two previously identified requirements:

1. Price freshness validation integrated into the combined eligibility pipeline.
2. A minimum requirement of 252 valid closing-price observations.

Add regression tests for data integrity and future-information invariance.

### 2. Baseline Price Freshness Audit

Reproduced the stale-price vulnerability in the existing combined universe function.

Test scenario:

- Selection date: 2021-04-15.
- ALPHA last available price: 2021-04-09.
- Liquidity observations: 60.
- Existing final eligibility: `True`.

The result demonstrated that a sufficient trailing liquidity window does not guarantee price freshness.

An initial test used an earlier stale-price cutoff that also caused the liquidity window to become incomplete. The test was corrected to isolate freshness from liquidity eligibility.

### 3. Integrated Price Freshness

Implemented:

`get_pit_tradable_universe_v2()`

The function incorporates a synthetic price-freshness check into price eligibility and recomputes the final eligibility decision.

Tests confirmed:

- Fresh ALPHA remained eligible.
- Stale ALPHA was excluded even with sufficient liquidity observations.

The freshness rule remains provisional and requires a real exchange trading calendar.

### 4. Minimum Valid-Price History

Implemented:

`get_pit_history()`

The function counts non-missing, positive closing-price observations dated on or before the selection cutoff.

Created an extended synthetic market dataset with:

- 300 pandas business dates.
- 1,300 total observations.
- ALPHA with 300 observations.
- GAMMA with 100 observations.

Verified the minimum-history boundary:

- 251 observations: ineligible.
- 252 observations: eligible.
- 300 observations: eligible.

The function rejects duplicate security-date records.

### 5. Combined Eligibility Version 3

Implemented:

`get_pit_tradable_universe_v3()`

The function combines historical security eligibility, price threshold, price freshness, liquidity, and minimum-history eligibility.

Results:

- ALPHA passed with 300 valid historical observations.
- GAMMA failed with only 100 valid historical observations.

An isolated test raised GAMMA's price to $50 while preserving its insufficient history.

GAMMA then passed price and liquidity checks but remained ineligible because of minimum history.

### 6. Regression Tests

Passed the following regression tests:

1. Future price and volume modifications after the historical cutoff did not change the historical eligibility results.
2. Stale ALPHA was excluded despite having sufficient liquidity and historical observations.
3. Duplicate security-date observations were rejected.

The future-data invariance test compared the complete returned DataFrames, not only final eligible ticker lists.

### 7. Research Interpretation

The new checks reduce specific data-integrity and point-in-time eligibility risks.

However, passing these synthetic tests does not establish production-grade historical backtest validity.

Remaining risks include:

- Exchange-calendar correctness.
- Data publication and availability timing.
- Missing or invalid trading observations.
- Historical security-master accuracy.
- Trading suspensions and delistings.
- Corporate-action adjustment and return reconstruction.

### 8. Files Updated

- `notebooks/02_pit_universe.ipynb`
- `docs/data_dictionary_v0.md`
- `docs/research_journal.md`

### 9. Next Steps

Week 2 Day 8 will focus on exchange trading calendars and data availability timing, with particular attention to the distinction between observation dates and model decision timestamps.