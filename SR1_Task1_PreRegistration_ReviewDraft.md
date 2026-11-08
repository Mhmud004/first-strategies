# SR-1 Task 1 --- Strategy Pre-registration

**Status:** Draft for reviewer approval. No strategy code has been run,
no backtest has been performed, and no performance results are reported.

**Purpose:** Pre-register three candidate strategies (A, B, and C)
before any strategy runs. This document records each hypothesis,
universe, rule, parameter grid, benchmark, and cost treatment. Items
marked **REVIEWER DECISION REQUIRED** are not specified by the supplied
SR-1 Addendum A or the available prior brief and must be confirmed
before the registration is considered signed off.

## 1. Scope and operating contract

This work is performed in the contributor's standalone
`first-strategies` repository. The contributor does not require access
to the trading repository. Strategy changes are handed to the
engineering lead through a reviewed change.

Until DL-1 real data is available, development is against
engineering-supplied synthetic bars in the platform's bar-file format.
Strategy code must read only that format so that switching to the real
point-in-time DL-1 export does not require strategy-code changes.

### 1.1 Strategy function contract

Each strategy is a pure function:

-   Input: bars available up to and including the close of day *t*.
-   Output: target weights by symbol for the next trading session, or an
    empty result when no positions should be held.
-   No access to a clock, network, filesystem, or other external state.
-   The same input history must produce the same weights, regardless of
    whether later bars exist.
-   Signals may use no information later than the close of day *t*;
    execution is for the next session.
-   Long only; no leverage or shorting; total target weights must be at
    most 1.
-   No same-day round trips.

### 1.2 Process and data safeguards

1.  Commit this pre-registration before running any strategy.
2.  Develop only on the development period and log every parameter
    variant attempted.
3.  Do not inspect, tune against, or rerun the hold-out. The reviewer
    performs the single hold-out run on the platform.
4.  A failed gate is a valid result and must be reported honestly.
5.  Report results only with the approved cost model applied; never show
    cost-free performance as the strategy result.
6.  Synthetic-bar development results must be clearly labelled synthetic
    and must not be represented as evidence of real-market performance.
7.  No strategy is approved for trading by this pre-registration.

## 2. Shared experimental choices

  -----------------------------------------------------------------------
  Item                                Pre-registered proposal / status
  ----------------------------------- -----------------------------------
  Decision time                       After the close of day *t*.

  Target weights                      Weights apply to the next session,
                                      not the signal day.

  Execution assumption                Next-session open, subject to
                                      confirmation that this matches the
                                      platform's approved execution
                                      convention. **REVIEWER DECISION
                                      REQUIRED**

  Costs                               Apply the platform's approved
                                      commission, spread, and slippage
                                      model to every result. Exact values
                                      and implementation must come from
                                      the platform/engineering lead.
                                      **REVIEWER DECISION REQUIRED**

  Benchmark                           Candidate-specific benchmark is
                                      defined below; benchmark execution
                                      and cost treatment must use the
                                      same approved conventions as the
                                      strategy.

  Development / hold-out split        Use the gate's official split. Do
                                      not choose or inspect the hold-out
                                      locally. **REVIEWER DECISION
                                      REQUIRED: identify split/gate
                                      specification**

  Bar source                          Engineering-supplied synthetic bars
                                      until DL-1 is available; then
                                      point-in-time DL-1 bars in the same
                                      schema.

  Corporate actions and delistings    Use the supplied data fields and
                                      official DL-1 treatment. Confirm
                                      adjusted-price and delisting-return
                                      semantics before real-data
                                      evaluation. **REVIEWER DECISION
                                      REQUIRED**

  Position constraints                Long only, no leverage, no shorts,
                                      weights sum to at most 1.

  Variant logging                     Record every parameter combination
                                      before execution; maximum grids are
                                      specified per candidate.
  -----------------------------------------------------------------------

## 3. Candidate A --- Trend on ETFs

### Hypothesis

A trend filter on a diversified set of liquid US ETFs, combined with
inverse-volatility weighting, may reduce drawdowns relative to a
buy-and-hold portfolio of the same ETF universe. Drawdown reduction is
the primary hypothesis; it is not a claim that the strategy will pass.

### Universe

Rule-based US-listed ETF universe covering: - Broad equity indices -
Equity sectors - Bonds - Gold

**Proposed selection rule:** At each scheduled universe review, select
the most liquid eligible ETFs in each role using trailing 60-session
median dollar volume, subject to at least one year of available history.

**Proposed counts:** 4 broad-index ETFs, 11 sector ETFs, 4 bond ETFs,
and 1 gold ETF.

**REVIEWER DECISION REQUIRED:** Confirm universe roles, counts,
liquidity threshold/ranking procedure, review schedule, and treatment of
a newly eligible or ineligible ETF. No hand-picked ticker list will be
introduced after seeing results.

### Signal and target-weight rule

At each scheduled rebalance: 1. An ETF is eligible to be held when its
close is above its own simple moving average over the selected trend
window. 2. For eligible ETFs, calculate inverse-volatility scores using
the selected trailing volatility window. 3. Convert scores into target
weights using the pre-registered denominator over the full fixed
universe; ineligible ETF allocations remain in cash. 4. Set all other
ETF weights to zero.

The precise denominator and handling of missing/insufficient history
must be confirmed before implementation. **REVIEWER DECISION REQUIRED.**

### Parameter grid

  Parameter                                   Values   Variants
  -------------------------------- ----------------- ----------
  Trend lookback (sessions)            150, 200, 250          3
  Volatility lookback (sessions)              20, 60          2
  Rebalance frequency                Weekly, monthly          2
  **Maximum variants**                                   **12**

### Benchmark

Buy-and-hold of the same eligible ETF universe, equal-weighted and
rebalanced using the same comparison schedule, with the same approved
cost assumptions.

### Pre-registered evaluation / failure conditions

The candidate does not support its hypothesis if any of the following
occurs: - The reduction in maximum drawdown versus the benchmark is less
than the proposed 20% relative reduction. - After-cost Sharpe is below
the benchmark's. - The conclusion reverses under the proposed 1.5x cost
stress. - Neighbouring pre-registered parameter settings produce
materially inconsistent conclusions. - The platform's approved
multiple-testing / deflated-result gate is not passed.

**REVIEWER DECISION REQUIRED:** Approve or replace the 20% drawdown
threshold, cost-stress multiplier, definition of materially
inconsistent, and multiple-testing gate. These are proposals, not
platform rules.

## 4. Candidate B --- Momentum on large-cap stocks

### Hypothesis

A cross-sectional momentum strategy that ranks large-cap stocks by
medium-term return while skipping the most recent month may outperform
an equal-weighted benchmark after costs. The candidate also tests
whether point-in-time universe membership and delisted names are handled
correctly.

### Universe

US large-cap stocks using the official point-in-time membership supplied
by DL-1, including names that later delist.

**REVIEWER DECISION REQUIRED:** Identify the exact DL-1 large-cap
universe/membership field and confirm how delisted securities and
delisting returns are represented. Do not substitute today's surviving
constituents for historical membership.

### Signal and target-weight rule

At each month-end: 1. Calculate each eligible stock's total return over
the selected lookback, excluding the most recent month. 2. Rank eligible
stocks cross-sectionally. 3. Select the top pre-registered percentage.
4. Assign equal target weights to selected stocks for the next monthly
holding period; all other weights are zero.

If a symbol becomes ineligible or leaves the point-in-time universe,
apply the platform's approved exit/eligibility convention. **REVIEWER
DECISION REQUIRED.**

### Parameter grid

  Parameter                                              Values   Variants
  ------------------------------------------- ----------------- ----------
  Lookback (months), excluding latest month            6, 9, 12          3
  Selected top group                                   10%, 20%          2
  Skip latest month                             1 month (fixed)          1
  Rebalance                                     Monthly (fixed)          1
  **Maximum variants**                                               **6**

### Benchmark

Equal-weighted buy-and-hold of the same point-in-time universe, with the
same approved cost assumptions.

### Pre-registered evaluation / failure conditions

The candidate does not support its hypothesis if any of the following
occurs: - Hold-out after-cost excess return versus the benchmark is zero
or negative. - Proposed cost drag exceeds 50% of gross excess return. -
The sign of excess return changes when delisted securities are correctly
included versus an explicitly labelled diagnostic that excludes them;
this is treated as a data-handling concern, not a pass. - The conclusion
reverses under the proposed 1.5x cost stress. - The platform's approved
multiple-testing / deflated-result gate is not passed.

**REVIEWER DECISION REQUIRED:** Approve or replace the 50% cost-drag
threshold, cost-stress multiplier, and multiple-testing gate.

## 5. Candidate C --- Short-term pullback

### Hypothesis

Buying liquid large-cap stocks after a sharp short-term decline, while
they remain above a long-term trend filter, may produce positive average
trade returns after realistic costs. This candidate is expected to be
especially sensitive to transaction costs; that is a hypothesis, not a
result.

### Universe

The same official point-in-time large-cap universe as Candidate B,
including delisted names. Signals are evaluated daily.

### Signal and target-weight rule

1.  Long-term trend condition: close above the 200-session simple moving
    average.
2.  Pullback condition: the stock's return over the selected short
    window ranks among the worst selected percentile of the eligible
    universe on that day.
3.  Entry: target a position for the next session when both conditions
    hold.
4.  Exit: target zero for the next session after the close first exceeds
    the 5-session simple moving average, or once the maximum holding
    period is reached, whichever occurs first.
5.  Sizing: equal weight, no more than 20 concurrent positions, long
    only, total weights at most 1.

**REVIEWER DECISION REQUIRED:** Confirm the exact return definition,
ranking/tie handling, re-entry rule, treatment of existing holdings that
remain in the signal set, and how the 20-position cap is applied when
more than 20 names qualify.

### Parameter grid

  Parameter                                           Values   Variants
  ----------------------------------- ---------------------- ----------
  Pullback lookback (sessions)                          3, 5          2
  Worst-return percentile                            5%, 10%          2
  Maximum holding period (sessions)                     3, 5          2
  Long-term trend filter                200 sessions (fixed)          1
  Rebound exit average                    5 sessions (fixed)          1
  **Maximum variants**                                            **8**

### Benchmark

Equal-weighted buy-and-hold of the same point-in-time universe, with the
same approved cost assumptions. Report average exposure alongside
results because the strategy may hold cash.

**REVIEWER DECISION REQUIRED:** Confirm whether the platform's D-11
benchmark rule applies to a strategy that may be flat by close.

### Pre-registered evaluation / failure conditions

The candidate does not support its hypothesis if any of the following
occurs: - Average net return per trade after costs is zero or
negative. - Proposed cost drag exceeds 50% of gross profit. - Hold-out
after-cost return is below the benchmark's. - It does not beat a matched
random-entry control using the same universe, holding duration, and
number of trades. - The conclusion reverses under the proposed 1.5x cost
stress. - The platform's approved multiple-testing / deflated-result
gate is not passed.

**REVIEWER DECISION REQUIRED:** Approve or replace the 50% cost-drag
threshold, cost-stress multiplier, matched-control construction, and
multiple-testing gate.

## 6. Pre-registration and variant log requirements

Before any strategy run, commit this document and obtain reviewer
sign-off on all required open decisions. The variant log must contain,
at minimum:

-   Candidate identifier (A, B, or C)
-   Exact parameter values
-   Unique variant identifier
-   Timestamp/commit identifier for the registered specification (not
    used as a strategy input)
-   Development run identifier and status
-   Confirmation that the run used only development data
-   Cost-model version
-   Any implementation deviation, with reviewer approval

Maximum permitted variants: A = 12, B = 6, C = 8. No additional
parameter combination or exploratory variation is permitted under this
registration. Any change requires a new reviewed pre-registration before
it is run.

## 7. No-look-ahead test plan

The implementation must include a deterministic prefix-invariance test
for each candidate:

1.  Select a date *t* in the available synthetic development bars.
2.  Run the strategy using only bars through the close of *t* and save
    its target weights.
3.  Run the same strategy against a longer dataset containing later
    bars, while requesting the weights for the same date *t*.
4.  Assert that the weights for *t* are identical (within a documented
    numeric tolerance).
5.  Repeat across multiple dates, including dates near the start/end of
    the available history and dates around universe membership changes
    where applicable.

Additional checks: - No signal uses a bar after *t*. - All target
weights apply to the next session. - Total weights are non-negative and
sum to at most 1. - No short positions or leverage. -
Empty/insufficient-history cases return an empty result or the agreed
safe output. - The same input produces the same output.

The engineering lead must confirm the synthetic bar schema and test
harness before implementation. This document does not claim that these
tests have been implemented or passed.

## 8. Reporting plan

For each candidate, the eventual development and reviewer-run hold-out
report should include the metrics required by the SR-1 gate, including
annual return and volatility, maximum drawdown, Sharpe ratio, turnover,
transaction-cost drag, worst month, hit rate, benchmark comparison,
exposure where relevant, capacity estimate, number of variants tried,
and the gate's multiple-testing assessment. All simulated results must
be labelled as simulated.

No hold-out performance is to be viewed or used for tuning by the
contributor.

## 9. Decisions required for sign-off

The supplied Addendum A does not specify the following values or
implementation details. The reviewer/engineering lead must confirm them
before this registration is treated as final:

1.  Exact standalone repository URL and required branch / PR workflow.
2.  Synthetic bar file schema, field names, calendar, and example
    fixture.
3.  Official DL-1 universe definitions and point-in-time membership
    fields for ETFs and large-cap stocks.
4.  Adjusted-price, dividend, delisting, and delisting-return
    conventions in DL-1.
5.  Approved execution convention and cost-model values/version.
6.  Official development/hold-out split and gate thresholds, including
    multiple-testing rule.
7.  D-3 capital constraints, D-11 benchmark treatment for flat-by-close
    strategies, and D-12 sign-off authority.
8.  Approval of all values marked proposed: ETF universe counts and
    selection procedure; 20% drawdown reduction; 50% cost-drag
    thresholds; 1.5x cost stress; cash return assumption if relevant;
    and control/robustness definitions.
9.  Candidate C tie-breaking, re-entry, and position-cap rules.

## 10. Current status

-   Pre-registration document drafted from the supplied SR-1 Addendum A
    and the existing candidate-strategy draft.
-   No strategy code run.
-   No backtest run.
-   No performance numbers generated.
-   No hold-out data accessed.
-   Awaiting reviewer decisions and sign-off before implementation
    begins.
