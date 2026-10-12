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