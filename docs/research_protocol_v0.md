# Equity Alpha Research — Research Protocol v0

Project: Leakage-Aware Cross-Sectional Equity Alpha Research  
Status: Draft  
Last updated: 2026-10-15

## 1. Research Objective

Investigate whether transparent price–volume signals contain out-of-sample information about future cross-sectional US equity returns, whether shallow nonlinear models add predictive value beyond simpler baselines, and whether any improvement remains economically meaningful after trading costs.

### 1.1 Research Hypotheses

**H1 — Signal Predictability**

Predefined price–volume signals contain positive and reasonably stable out-of-sample information about the cross-sectional ranking of future equity returns.

Primary evidence: out-of-sample Rank IC, evaluated across dates with appropriate uncertainty estimates.

**H2 — Incremental Value of Nonlinear Models**

A shallow nonlinear boosting model provides incremental out-of-sample cross-sectional predictive value relative to a transparent signal composite and a regularized linear baseline (Ridge).

Primary evidence: paired differences in out-of-sample Rank IC on common evaluation dates and securities.

**H3 — Economic Relevance**

Any incremental predictive value translates into economically meaningful portfolio performance after accounting for turnover, transaction costs, and relevant short-selling costs.

Primary evidence: cost-adjusted portfolio returns, risk, turnover, and sensitivity to transaction-cost assumptions.

Failure to support any hypothesis will be reported as a valid research outcome.

## 2. Information Timing Protocol

### 2.1 Signal Frequency

- Signal frequency: Weekly.
- Signal observation date: The final regular trading day of each calendar week on the US equity market.
- Data frequency: Daily OHLCV.
- All signal inputs must be available before the model decision time.

### 2.2 Feature Cutoff

Feature cutoff is the end of the signal observation trading session.

Only information observed and available at or before this cutoff may be used to construct features.

For example, a signal computed after Friday's close may use that Friday's confirmed OHLCV data, but not prices or volumes from the following trading day.

### 2.3 Decision Time

The model generates forecasts and determines the intended portfolio after the latest signal-date OHLCV data have become available.

Decision time must precede execution time.

No information published or observed after the decision time may influence the corresponding forecasts.

### 2.4 Entry / Execution Time

The assumed entry execution time is the regular-session close of the first valid US equity trading day after the signal observation date.

The research will initially use closing prices as a simplified execution-price assumption.

Trading costs, execution feasibility, security-specific trading restrictions, and potential deviations from closing prices will be considered separately.

### 2.5 Exit Time

The assumed exit time for each weekly prediction sample is the regular-session close of the first valid US equity trading day following the next weekly signal observation date.

The holding period is therefore approximately one trading week, with the exact dates determined by the US exchange trading calendar.

### 2.6 Target Return

For security (i) and weekly signal date (t), the target is the return over the specified entry-to-exit interval:

$$
y_{i,t}=\frac{P_{i,\mathrm{exit}}}{P_{i,\mathrm{entry}}}-1
$$

The final price/return convention must consistently account for dividends, splits, delistings, and other relevant corporate actions.

Raw closing prices must not automatically be interpreted as total-return prices.

### 2.7 Label Availability

A training label becomes available only after the corresponding exit execution price and required corporate-action information can be confirmed.

At each model training cutoff, observations with labels that have not yet matured must be excluded.

This rule also applies to expanding-window and walk-forward validation.

### 2.8 Trading Calendar

Signal observation, entry, and exit dates must be derived from a valid US equity exchange calendar.

Calendar holidays must not be treated as trading sessions.

If a security cannot be traded at the required execution time, a separate eligibility and missing-execution policy must be applied.

## 3. Research Universe & Eligibility

### 3.1 Target Market

The research focuses on US-listed common equities traded on NYSE, Nasdaq, and NYSE American.
ETFs, preferred shares, and other non-common-equity instruments will be excluded, subject to the security classification available from the selected data provider.

### 3.2 Point-in-Time Universe Construction

The eligible security universe must be reconstructed using information available at each historical selection date.

A security must not be excluded from a historical universe solely because it was subsequently delisted, acquired, or otherwise ceased trading.

Similarly, securities that had not yet been listed at the historical selection date must not be included.

Historical security identifiers, listing dates, delisting dates, and relevant corporate actions must be handled consistently.

### 3.3 Preliminary Eligibility Filters

Candidate filters:
- Minimum price: $5 per share.
- Minimum median daily dollar volume: $10 million over the preceding 60 trading sessions.
- Minimum available price history: 252 trading sessions.
- Security type: eligible US-listed common equity.
- Liquidity and price measurements must use information available by the applicable selection cutoff.

Dollar volume is provisionally defined as:

$$
DV_{i,d}=P_{i,d}\times V_{i,d}
$$

where the price and volume definitions must be consistent with the selected vendor's corporate-action adjustment conventions.

The eligibility rules will be finalized after the historical data provider and security master have been audited.

### 3.4 Universe Reconstitution

The primary design is to reconstitute the eligible universe monthly using historical information available by the selection cutoff.
Weekly forecasts will be generated for securities in the applicable eligible universe.
The selection date, universe effective date, and security eligibility records must be stored explicitly.

### 3.5 Survivorship Bias and Delisting Treatment

The research must seek coverage of active and historically delisted securities.
Delisted securities must remain eligible in earlier periods when they satisfied the historical selection criteria.

Return calculations must account for relevant delisting events and corporate actions.
If the final data provider does not support adequate historical universe reconstruction or delisting-return treatment, the research scope and limitations must be revised before treating the results as survivorship-bias-controlled.

### 3.6 Pending Decisions

The following rules remain provisional:
- Historical data provider and identifier convention.
- Exact common-share classification and exchange identifiers.
- Eligibility-filter reference prices and adjustment conventions.
- Missing-price and trading-halt policies.
- Delisting-return and portfolio-exit conventions.
- Final price and liquidity thresholds.

## 4. Out-of-Sample Validation Protocol

### 4.1 Development and Final Holdout Periods

The target research window is approximately 2005–2025, subject to data availability and an adequate feature warm-up period.

The proposed final holdout period is 2024–2025.

All model selection, feature selection, and research-protocol tuning must be completed using the development period before the final holdout is evaluated.

The final holdout must not be repeatedly reused for model or hyperparameter selection.

### 4.2 Expanding Walk-Forward Validation

The primary validation approach will use chronological expanding-window training and subsequent out-of-sample evaluation.

At each model fitting date:

- Training observations must precede the relevant evaluation period.
- All training labels must already be available at the fitting cutoff.
- Model preprocessing, imputation, scaling, and hyperparameter fitting must use training-eligible observations only.

Predictions must be generated without access to future returns or future universe membership.

Evaluation observations must be grouped by date rather than randomly split by individual stock-day rows.

### 4.3 Label Maturity and Leakage Controls

A historical observation becomes eligible for training only after its complete target return can be computed from the specified entry and exit interval.

Overlapping label intervals and temporal dependence must be considered when defining training and validation boundaries.

The implementation must explicitly audit whether each training label was available before the model fitting time.

### 4.4 Model Comparison

The planned model hierarchy is:

1. Transparent rank-based signal composite.
2. Regularized linear model (Ridge).
3. Shallow nonlinear boosting model.

The primary predictive metric is out-of-sample cross-sectional Rank IC.

The incremental predictive contribution of boosting will be evaluated relative to simpler baselines using paired comparisons over common evaluation dates and securities.

### 4.5 Statistical Inference

Statistical uncertainty will be evaluated using procedures appropriate for dependent financial observations.

Candidate methods include date-level or calendar-block bootstrap inference.

Multiple testing adjustments will be considered for a limited, predeclared family of core hypotheses.

Exploratory results must be distinguished from confirmatory results.

### 4.6 Protocol Freeze and Final Evaluation

The research protocol, final feature set, model-selection rules, and evaluation procedure must be frozen before the final holdout is inspected.

The provisional freeze target is December 13, 2026.

Final holdout results must be reported even if they do not support the original research hypotheses.

Any post-holdout exploratory modifications must be clearly identified and must not be presented as independent confirmatory evidence.

### 4.7 Pending Decisions

The following specifications remain to be finalized:

- Minimum initial training-window length.
- Exact retraining frequency.
- Detailed walk-forward folds.
- Hyperparameter search budget.
- Purging or embargo requirements for overlapping labels.
- Final bootstrap configuration.
- Final development and holdout boundaries if data coverage differs from the target.

## 5. Open Design Decisions

The following items remain to be finalized:

Historical universe definition and survivorship-bias treatment.

Primary data provider and corporate-action conventions.

Exact forecast features and baseline models.

Model retraining frequency and walk-forward validation.

Transaction costs and portfolio construction assumptions.

Statistical inference and final holdout procedure.