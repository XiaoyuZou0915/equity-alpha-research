# Equity Alpha Research — Data Dictionary v0

Status: Draft

## 1. Data Source

- Exploratory provider: Yahoo Finance via yfinance
- Current sample: AAPL, MSFT, JPM, XOM, WMT
- Date range: 2020-01-02 to 2024-12-31
- Frequency: Daily
- Research use: Exploratory data-quality checks only
- Production research provider: To be determined

## 2. Field Definitions

| Field | Preliminary Definition | Audit Requirement |
|---|---|---|
| Open | Opening price | Confirm adjustment convention |
| High | Daily high price | Confirm adjustment convention |
| Low | Daily low price | Confirm adjustment convention |
| Close | Closing price; split-adjusted in observed Yahoo sample | Verify vendor convention |
| Adj Close | Corporate-action-adjusted closing price | Verify dividend, split and adjustment conventions |
| Volume | Daily reported share volume | Verify split adjustments and share units |
| Dividends | Cash dividend event amount per share | Verify ex-date and adjustment basis |
| Stock Splits | Stock split event ratio | Verify event date and ratio convention |

## 3. Return Definitions

**Price Return**

Daily percentage change in the split-adjusted Close series.

**Adjusted Return**

Daily percentage change in Adj Close.

**Dividend-Inclusive Return**

For compatible price and cash-dividend units:

$$
r_{i,t}=\frac{P_{i,t}+D_{i,t}}{P_{i,t-1}}-1
$$

This is a simplified one-period holding return and requires further validation of corporate-action conventions.

## 4. Point-in-Time Considerations

- Historical adjusted prices may incorporate corporate actions that occurred after the historical observation date.
- Feature construction must not unintentionally use future adjustment information.
- Historical universe membership must reflect eligibility at the relevant date.
- Delisted securities and delisting returns must be investigated before formal backtesting.

### 4.1 Point-in-Time Feature Construction Rules

**Observation time vs. availability time**

A historical observation date does not necessarily indicate when a value became available to researchers or traders.

Historical data availability and subsequent vendor revisions must be considered when reconstructing features.

**Corporate-action adjustment**

Price-based features must use a consistent adjustment convention.

For a momentum feature:

$$
MOM_{i,t}^{(20)}=\frac{P_{i,t}}{P_{i,t-20}}-1
$$

A common multiplicative adjustment applied to both prices does not change this ratio.

However, absolute price thresholds must reflect the intended historical share-price convention. A future split-adjusted historical price must not be mistaken for the quoted price observed at the original date.

**Universe eligibility**

Historical price thresholds, listing status, security classifications and liquidity criteria must be evaluated using eligible information available by the selection cutoff.

The research must not select historical securities based on their later survival or current index membership.

**Dividend and return conventions**

Price returns and total returns must be distinguished.

Predictive labels and portfolio evaluation must use explicitly documented economic return conventions.

**Implementation requirements**

Before formal backtesting, the data pipeline must:

- Document each source field's adjustment convention.
- Distinguish observation date from availability date where relevant.
- Audit split and dividend events.
- Check historical security identifiers and delisting coverage.
- Ensure price and volume units are compatible when calculating dollar volume.
- Record unresolved point-in-time limitations.

These rules are provisional until the production data provider has been audited.

## 5. Known Limitations

Yahoo Finance is currently used only for exploratory research.

The five-stock sample is not a survivorship-bias-controlled historical universe.

Historical adjusted-price conventions, delisting coverage and point-in-time reproducibility have not yet been fully audited.

## 6. Production Data Source Audit

### 6.1 CRSP via WRDS

Status: Access not confirmed.

The researcher has a Cornell Alumni account but has never registered an independent WRDS account.

Cornell's published access policies restrict WRDS access; alumni access is not automatically available.

CRSP remains a preferred research-grade source, but data access must be verified before implementation.

### 6.2 Sharadar Equity Prices (SEP)

**Status:** Candidate alternative; access and cost not yet verified.

The published dataset documentation describes:

- Historical US daily equity prices.
- Coverage of active and delisted securities.
- Multiple corporate-action adjustment conventions.
- Security reference information and corporate actions.

Required follow-up audits:

- Subscription and licensing costs.
- Historical security identity and universe reconstruction.
- Delisting-event and return handling.
- Point-in-time adjustment availability.
- Data coverage, revisions and missing values.

#### Preliminary Vendor Documentation Audit

Documentation reviewed: October 17, 2026.

**Verified from published documentation:**

- SEP covers daily prices for active and delisted US securities, with history extending to approximately 1998.
- `closeunadj`: unadjusted daily closing price.
- `close`: split-adjusted daily closing price.
- `closeadj`: closing price adjusted for splits, dividends, and spinoffs.
- `volume`: split-adjusted trading volume.
- `lastupdated`: date of the latest record update.
- The companion corporate-actions dataset includes split, dividend, listing, and delisting events.
- The security reference dataset provides permanent identifiers (`permaticker`).

**Unverified requirements:**

- Complete historical point-in-time availability.
- Exact delisting-return and terminal-price coverage.
- Corporate-action adjustment reproducibility.
- Historical universe and security-classification accuracy.
- Subscription price and permitted research usage.

**Access decision:** Candidate provider only. Do not purchase or implement production ingestion yet.

**References:**

- [Sharadar Stock Prices](https://sharadar.com/docs/stocks)
- [Sharadar Corporate Actions](https://sharadar.com/docs/actions)
- [Nasdaq SEP Documentation](https://data.nasdaq.com/databases/SEP/documentation)

### 6.3 Yahoo Finance

Status: Exploratory only.

The existing five-stock dataset is suitable for code prototyping and corporate-action sanity checks, but not for final survivorship-bias-controlled research conclusions.

### 6.4 Decision

Do not finalize the production data provider until historical coverage, survivorship-bias treatment, point-in-time properties and access costs have been evaluated.

Avoid developing a full production backtest based on an unverified data source.