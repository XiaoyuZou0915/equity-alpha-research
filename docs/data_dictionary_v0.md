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

## 7. Security Master and Point-in-Time Universe

### 7.1 Purpose

The Security Master provides a stable security identity and historical eligibility information for constructing a point-in-time equity universe.

Ticker symbols must not be treated as permanent unique security identifiers.

The initial implementation uses synthetic data to validate universe selection logic before production data integration.

### 7.2 Prototype Security Master Schema

| Field | Data Type | Description |
|---|---|---|
| security_id | Integer | Stable internal security identifier |
| ticker | String | Security ticker in the simplified prototype |
| listing_date | Date | First eligible listing date |
| delisting_date | Date / Null | Last trading date under the prototype convention |
| exchange | String | Exchange classification |
| share_type | String | Security classification |

The prototype assumes ticker, exchange, and share type remain constant.

Production data must support historical changes in these attributes and distinguish effective dates from information availability dates.

### 7.3 Universe Eligibility Rules

The prototype function `get_pit_universe(security_master, as_of_date)` applies the following rules:

1. `listing_date <= as_of_date`
2. `delisting_date` is missing or `delisting_date >= as_of_date`
3. Exchange is NYSE, NASDAQ, or NYSE American
4. Share type is COMMON

These rules are evaluated using synthetic security records.

Price, liquidity, trading-halt, and historical metadata availability filters are not yet implemented.

### 7.4 Historical Universe Examples

| Selection Date | Eligible Securities |
|---|---|
| 2015-06-30 | ALPHA, BETA |
| 2019-06-30 | ALPHA |
| 2021-06-30 | ALPHA, GAMMA |

BETA is included in 2015 even though it delists in 2018.

GAMMA is excluded before its 2020 listing.

DELTA is excluded because it is an ETF.

EPSILON is excluded because it trades on OTC.

### 7.5 Automated Testing

The prototype includes assertions covering:

- Historical universe membership.
- Inclusion of securities that delist later.
- Exclusion of securities before their IPO.
- Exclusion of ETFs and OTC securities.
- Listing-date boundary behavior.
- Delisting-date boundary behavior.

A separate experiment demonstrates survivorship bias by removing delisted securities before historical universe construction.

These are synthetic unit tests, not proof of production data quality.

### 7.6 Production Requirements

Before production use, the system must support:

- Permanent security identifiers and historical identifier mappings.
- Historical ticker, exchange, and share-type changes.
- Security metadata effective dates and availability dates.
- Historical price and liquidity eligibility rules.
- Delisting events and terminal-return treatment.
- Missing data and trading-halt policies.
- Automated data integrity and leakage checks.

**Status:** Prototype only. Not approved for production backtesting.

## 8. Point-in-Time Price and Liquidity Eligibility

### 8.1 Purpose

Extend the synthetic point-in-time security universe prototype with historical price and liquidity eligibility filters.

Implementation notebook:

`notebooks/02_pit_universe.ipynb`

The implementation uses synthetic data only and is not approved for production backtesting.

### 8.2 Synthetic Daily Market Data

The prototype contains 80 synthetic business dates and five securities, giving 400 security-date records.

| Field | Data Type | Description |
|---|---|---|
| date | Date | Synthetic market observation date |
| security_id | Integer | Stable security identifier |
| close | Float | Synthetic closing price |
| volume | Integer | Synthetic share trading volume |
| dollar_volume | Float | Closing price multiplied by volume |

The synthetic date index uses pandas business days rather than a complete US exchange calendar.

The dataset intentionally includes price records for BETA after its recorded delisting date to test whether the Security Master prevents invalid universe membership.

### 8.3 Price Eligibility

The provisional minimum price is $5 per share.

A security passes the price filter if its latest available closing price at or before the selection cutoff is at least $5.

The prototype uses:

`get_pit_price_universe()`

The actual production implementation must use a historically valid price convention and enforce price freshness and availability requirements.

### 8.4 Dollar Volume

Daily dollar volume is defined as:

$$
DV_{i,d}=P_{i,d}\times V_{i,d}
$$

The selected price and volume fields must use compatible share units.

For the synthetic prototype, daily prices and volumes are constant for each security.

### 8.5 Median Dollar Volume

The liquidity metric is the median of the most recent 60 available daily dollar-volume observations at or before the selection cutoff:

$$
MDV_{i,t}^{(60)}
=
\operatorname{Median}
\left(DV_{i,t-59},\ldots,DV_{i,t}\right)
$$

The provisional threshold is $10 million.

The function `get_pit_liquidity()` requires 60 valid observations.

If fewer than 60 observations are available, the metric is set to missing and the security fails the liquidity filter.

The current implementation does not verify that these observations correspond to 60 consecutive exchange trading sessions.

### 8.6 Combined Eligibility

The function `get_pit_tradable_universe()` combines:

1. Historical listing and delisting eligibility.
2. Exchange and common-share eligibility.
3. Minimum price eligibility.
4. Minimum median dollar-volume eligibility.

A security is eligible only when all required conditions are satisfied.

For the synthetic example on 2021-04-15:

| Security | Price | Median Dollar Volume | Final Eligibility |
|---|---:|---:|---|
| ALPHA | $100 | $20 million | Eligible |
| GAMMA | $4 | $20 million | Ineligible |

BETA remains excluded because it had already delisted, despite the presence of synthetic price records.

### 8.7 Automated Tests

Seven automated test groups passed:

1. Historical security eligibility.
2. Combined price and liquidity eligibility.
3. Dollar-volume calculation.
4. Complete 60-observation windows.
5. Exclusion of insufficient-history securities.
6. Invariance of historical results to future price modifications.
7. Exclusion of previously delisted securities.

These tests demonstrate the behavior of the synthetic implementation but do not establish complete point-in-time integrity for a production data source.

### 8.8 Known Limitations

- No actual US exchange trading calendar.
- No explicit data availability timestamps.
- No price staleness or trading-halt checks.
- No validation of consecutive exchange trading sessions.
- No historical exchange or share-classification changes.
- No production corporate-action adjustment reconstruction.
- No delisting-return or execution-price handling.
- No production data provider integration.

**Status:** Synthetic research infrastructure prototype; production validation pending.

## 9. Week 1 Integration Audit and Engineering Backlog

### 9.1 Audit Scope

The Week 1 integration review compares:

- `docs/research_protocol_v0.md`
- `docs/data_dictionary_v0.md`
- `notebooks/02_pit_universe.ipynb`

The purpose is to identify differences between research requirements and the current synthetic Python implementation.

### 9.2 Requirements vs. Implementation

| Requirement | Current Implementation | Status |
|---|---|---|
| Historical listing and delisting eligibility | `get_pit_universe()` | Synthetic prototype implemented |
| Minimum share price of $5 | `get_pit_price_universe()` | Synthetic prototype implemented |
| Median dollar volume of at least $10m | `get_pit_liquidity()` | Synthetic prototype implemented |
| Complete 60-observation liquidity window | Required by the liquidity function | Implemented for synthetic data |
| Minimum 252-session price history | No separate minimum-history check | Not implemented |
| Monthly universe reconstitution | Functions accept an individual selection date | Not implemented |
| Price freshness | Separate `add_price_freshness_check()` function | Not integrated |
| Exchange trading calendar | Uses pandas business dates | Not implemented |
| Data availability timestamps | Uses observation dates only | Not implemented |
| Historical ticker and exchange changes | Static synthetic attributes | Not implemented |
| Delisting-return handling | No return reconstruction | Not implemented |

### 9.3 Engineering Backlog

#### P0 — Data Integrity and Leakage Controls

- Integrate price freshness checks into the main eligibility pipeline.
- Reject or quarantine invalid market observations before computing eligibility.
- Specify a minimum history requirement independent of the liquidity window.
- Validate historical observation dates against the applicable exchange calendar.
- Define data availability cutoffs and prevent future information leakage.

#### P1 — Historical Universe Improvements

- Implement monthly universe reconstitution.
- Support historical ticker, exchange, and share-type changes.
- Detect missing or stale market observations.
- Validate consecutive trading-session coverage in rolling liquidity windows.
- Add structured data-quality reports with security IDs, dates, and issue codes.

#### P2 — Production Data Integration

- Confirm access to a suitable research-grade data provider.
- Map provider security identifiers to stable internal IDs.
- Audit corporate-action adjustment conventions.
- Implement delisting-event and terminal-return handling.
- Validate point-in-time historical universe reconstruction.

### 9.4 Data Quality Findings

The synthetic data quality experiments demonstrated:

1. Duplicate security-date records can be detected and rejected.
2. Missing, zero, and negative prices should be distinguished from valid prices that fail eligibility thresholds.
3. The original price eligibility function may accept stale prices.
4. A separate freshness check can reject stale prices, but is not yet integrated into the main function.
5. Passing synthetic unit tests does not establish production-level point-in-time integrity.

### 9.5 Week 1 Completion Criteria

Week 1 completion means the exploratory data audits, synthetic universe prototype, and initial automated tests have been developed and reviewed.

It does not mean the production data pipeline is complete.

**Status:** Synthetic research infrastructure prototype; production validation pending.

## 10. Week 2 Day 7 — PIT Eligibility Hardening

### 10.1 Objective

Strengthen the Week 1 synthetic point-in-time (PIT) universe prototype by integrating:

1. Price freshness validation.
2. Minimum 252-observation valid-price history.
3. Regression tests for future-data invariance, stale prices, and duplicate market observations.

These changes remain part of a synthetic research prototype, not a production-ready historical equity universe.

### 10.2 Price Freshness

Function: `get_pit_tradable_universe_v2()`

The function reuses the historical universe, price, and liquidity checks from `get_pit_tradable_universe()`.

It introduces:

- `price_fresh`: whether the latest available price observation date matches the selection date.
- Updated `price_eligible`: the original price eligibility result combined with `price_fresh`.
- Recomputed `eligible`: price eligibility combined with liquidity eligibility.

For the current synthetic dataset:

```python
price_fresh = (latest_price_date == as_of_date)
```

This is a provisional rule.

It is not yet based on the actual exchange trading calendar, trading suspensions, or market-data availability timestamps.

The baseline audit demonstrated that the Week 1 combined function could accept ALPHA with a stale price dated 2021-04-09 at a selection date of 2021-04-15, despite satisfying the 60-observation liquidity requirement.

The v2 function successfully excluded this stale observation.

### 10.3 Minimum History

Function: `get_pit_history()`

Parameters:

- `daily_prices`: historical market observations.
- `as_of_date`: point-in-time cutoff.
- `min_history`: minimum required valid-price observations; default 252.

Output fields:

| Field | Definition |
|---|---|
| `security_id` | Stable synthetic security identifier |
| `history_count` | Number of valid closing-price observations dated on or before `as_of_date` |
| `history_eligible` | Boolean indicating whether `history_count >= min_history` |

A valid closing price must be non-missing and strictly positive.

Duplicate security-date records are rejected before calculating history counts.

The current implementation counts valid historical observations, not necessarily consecutive exchange trading sessions.

It does not yet verify exchange-calendar coverage, historical listing boundaries, or completeness of the preceding 252 trading sessions.

### 10.4 Combined Eligibility Version 3

Function: `get_pit_tradable_universe_v3()`

The function combines:

1. Historical security eligibility from the existing upstream universe logic.
2. Minimum price threshold.
3. Price freshness.
4. Median dollar-volume liquidity requirement.
5. Minimum valid-price history.

The function joins historical observation counts using `security_id`, with a one-to-one merge validation.

Missing history counts are treated as zero, and missing history-eligibility values are treated as `False`.

The final eligibility decision is:

```python
eligible = (
    price_eligible
    & liquidity_eligible
    & history_eligible
)
```

The `price_eligible` field in v3 already incorporates the synthetic price-freshness check from v2.

The historical security-master filters are applied upstream.

### 10.5 Synthetic Test Data

An extended synthetic dataset was created using 300 pandas business dates.

The dataset contains 1,300 security-date observations:

| Security | Observations | Historical eligibility role |
|---|---:|---|
| ALPHA | 300 | Sufficient history |
| BETA | 300 | Historical security-master exclusion |
| GAMMA | 100 | Insufficient history |
| DELTA | 300 | Security-type exclusion |
| EPSILON | 300 | Exchange exclusion |

The use of `pd.bdate_range()` is a synthetic convenience and does not establish NYSE trading-calendar validity.

The synthetic dataset deliberately includes observations whose relationship with listing and delisting history is not production-realistic.

### 10.6 Tests Completed

The following tests passed during Day 7:

- Baseline stale-price vulnerability reproduction.
- Clean-price and stale-price comparison using v2.
- Minimum-history boundary tests: 251, 252, and 300 observations.
- Combined eligibility v3 test.
- Independent minimum-history failure test using GAMMA.
- Future-data invariance after modifying later prices and volumes.
- Stale-price rejection with sufficient liquidity and history.
- Duplicate security-date record rejection.

The current tests demonstrate expected behavior for the constructed synthetic scenarios.

They do not establish full production-grade data integrity or the absence of all forms of look-ahead bias.

### 10.7 Remaining Engineering Backlog

#### High Priority

- Replace calendar-date equality with exchange-session-aware freshness validation.
- Define observation availability relative to the actual decision timestamp.
- Validate minimum history using historical trading-session coverage.
- Integrate consistent missing-price, invalid-price, and invalid-volume handling into the main eligibility pipeline.
- Add explicit historical security-master validity checks.

#### Medium Priority

- Implement monthly universe reconstitution.
- Handle historical ticker, exchange, share-class, and listing-status changes.
- Define behavior for trading suspensions and prolonged price staleness.
- Separate eligibility observations from return and execution price series.
- Add structured data-quality error reports.

#### Data Provider Dependencies

- Confirm access to a suitable historical equity data provider.
- Verify corporate-action adjustment conventions.
- Obtain reliable historical security identifiers and metadata.
- Audit delisting events and terminal returns.

### 10.8 Implementation Status

Week 2 Day 7 completed the synthetic integration of price freshness and minimum valid-price history into combined PIT eligibility.

The implementation is an intermediate research prototype.

**Production backtesting remains blocked by unresolved point-in-time data, calendar, availability, and historical security metadata requirements.**