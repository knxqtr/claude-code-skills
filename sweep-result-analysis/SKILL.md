---
name: sweep-result-analysis
description: Use when analyzing component sweep, param sweep, exit sweep, walk-forward, or holdout results, or when selecting configs for promotion to the next pipeline stage. Provides stage-specific analysis frameworks, policy gates, and promotion shortlist formats for all 5 pipeline stages.
---

# Sweep Result Analysis

Standardized analysis and promotion formats for each pipeline stage.

## When to Use

- After any sweep completes and results need analysis
- When selecting configs to promote to the next stage
- When comparing sweep results across families or versions

## General Principles (All Sweeps)

1. Verify every claim with code -- do not assume
2. Check column sign conventions before applying gates (e.g. max_drawdown may be positive magnitude)
3. Funnel width policy: comp and param sweep are WIDE (threshold-based, no family caps). Exit sweep is the NARROWING POINT (family + plateau analysis). WFT is WIDE again (~200 configs, trust DSR for multiple testing correction).
4. Ranking by stage: comp/param use terminal_reach_40. Exit sweep uses total_r per family. WFT uses calmar_r (total_r / max_dd) -- only WFT has trustworthy OOS drawdown.
5. Label robustness: ROBUST / MODERATE / FRAGILE with evidence
6. Always show family-level breakdown, not just global top-N
7. Note any estimated values or missing columns
8. PF gate is volume-adjusted at WFT: < 500 trades PF >= 1.2, 500-2000 PF >= 1.15, > 2000 PF >= 1.1

---

## Stage 1: Component Sweep Analysis

### Purpose

Rough screen across trigger/filter combinations. NOT the final judge of strategy quality. Answers: does this family deserve deeper work?

### Policy Gates

All must pass before ranking:
- Resolved signals >= 400
- Per-coin minimum: >= 20 resolved signals per counted coin
- E-ratio > 1.0 (conservative: ci_lo > 1.0)
- Directional accuracy: p < 0.01
- Unresolved fraction <= 10%
- Coin breadth >= 30% of eligible coins

### Ranking

Primary: terminal_reach_40 (current implementation) or hit-rate curve surplus above breakeven.
Tie-breaks: coin_breadth > signal_count > per-coin min E-ratio.

Do NOT rank by E-ratio alone or backtest expectancy from fixed exits.

### Analysis Framework (6 sections)

1. **Landscape:** total configs, passing gates, passing per trigger family
2. **Per-family top 5** by primary ranking metric. Show: trigger, filters, terminal_reach_40, speed_bonus, e_ratio, ci_lo, coin_breadth, signal_count, directional_accuracy, unresolved_fraction
3. **Hit-rate curve shape:** for top families, show hit rates at 1.0R/1.2R/1.4R/1.6R/2.0R. Note if curve is broad/stable or concentrated in one R zone
4. **Coin breadth distribution:** how many coins pass per family? Flag families where breadth is marginal (30-40%)
5. **Signal volume:** per-family signal counts. Flag families near the 400 minimum
6. **Cross-family summary table:**

| Family | Trigger | tr40 | speed | e_ratio | ci_lo | breadth | signals | unresolved% | curve_shape |

### Promotion Shortlist

- Policy says widen the funnel, prefer threshold-based over narrow top-N
- Preserve diversity across trigger and filter families
- Note whether top configs are broad/stable or single-zone

| # | Family | Trigger | Filters | tr40 | ci_lo | breadth | signals | Why |
|---|--------|---------|---------|------|-------|---------|---------|-----|

Every config must have a "Why" -- one-line reason for inclusion:
- "Family best, broad hit-rate curve across 1.0-2.0R"
- "Different filter family, preserves diversity"
- "Near-threshold gate, worth deeper param testing"

Also include exclusion rationale for families NOT promoted.

---

## Stage 2: Param Sweep Analysis

### Purpose

Refine trigger internals for families that passed component sweep. Answers: which parameter regions are strong and stable?

### Policy Gates

Same as component sweep: signals >= 400, E-ratio > 1.0, directional accuracy p < 0.01, unresolved <= 10%, coin breadth >= 30%.
Conservative: ci_lo >= 1.0.

### Ranking

Same 4-level tuple: terminal_reach_40 > speed_bonus > coin_breadth > signal_count.

### Key Policy Rules

- Do NOT treat single top point as winner
- Note parameter plateaus vs isolated maxima (informational, for exit sweep context)
- Exact tradeability belongs to exit sweep, not here
- Wide funnel: promote all configs that pass gates, threshold-based not family-capped
- Do NOT cap per-family N at this stage -- family curation happens at exit sweep
- Plateau analysis is informational here (document it), but do not use it to restrict promotion

### Analysis Framework (7 sections)

1. **Landscape:** total configs, passing gates per family
2. **Per-family top 5** by ranking tuple. Show: trigger_params (extracted individually: near_threshold, min_approach, min_bounce, cross_side_limit, fib_threshold), terminal_reach_40, e_ratio, ci_lo, coin_breadth, signal_count
3. **Plateau analysis per family:** within each family, count configs within 0.01 of best tr40. Are top configs clustered on one param point or spread across neighboring values? Flag sharp isolated peaks vs flat regions
4. **Parameter sensitivity:** which params drive the most variation? Compare performance across near_threshold, min_approach, min_bounce, cross_side_limit values
5. **ci_lo gate impact:** how many configs pass ci_lo >= 1.0 per family? Any families that pass plain e_ratio > 1.0 but fail ci_lo?
6. **Signal volume vs quality tradeoff:** scatter of signal_count vs tr40 per family. Note configs with high signals but lower ranking (potential volume plays for later)
7. **Cross-family summary table:**

| Family | Best tr40 | ci_lo | breadth | signals | plateau_size | plateau_width | sharp_peak? |

### Promotion Shortlist

Wide funnel: promote all gate-passers in principle. In practice, compute budget constrains to ~40-50 configs for exit sweep. Selection approach:

1. Top 1 per gate-passing family (ensures every hypothesis gets tested)
2. Remaining slots to strongest families (by best tr40) for param depth -- 2-3 per strong family
3. Total ~40-50 after dedup

This is still family-aware, but the key difference from old policy: no family gets shut out entirely, and strong families earn extra slots on merit rather than being hard-capped.

| # | Family | Trigger Params | tr40 | ci_lo | breadth | signals | Why |
|---|--------|---------------|------|-------|---------|---------|-----|

Note plateau information (size, robustness) for each family as context for exit sweep, but do not use it to restrict promotion at this stage.

Always include:
- Total gate-passing configs and families
- How many families got 1 slot vs 2-3 slots and why
- Plateau observations (informational, not restrictive)
- Exclusion rationale for configs that failed gates
- Caveats for configs near gates (marginal ci_lo, borderline breadth)

---

## Stage 3: Exit Sweep Analysis

### Purpose

First stage that decides real tradeability using full simulated results. Answers: which exit structures fit the signal? Is the winner robust across nearby exits?

### Policy Gates

- trades >= minimum (typically 500)
- net_expectancy_primary > 0 (must be net positive after fees)
- profit_factor >= 1.1
- max_drawdown <= threshold (column stores positive magnitude, use <=)

### Key Policy Rules

- This is the NARROWING POINT in the pipeline -- family and plateau curation starts here
- Mandatory exit robustness review: inspect neighboring SL/RR/exit settings
- Mark winners as ROBUST or FRAGILE based on how neighbors perform
- Promote multiple configs from robust neighborhoods, not just single highest row
- Preserve diversity across exit variants and filter families
- Rank by total_r within families (NOT calmar -- in-sample DD is not trustworthy for ranking)
- Target ~200 configs to WFT: top 15-25 per family by total_r, with plateau/robustness check
- DSR at WFT self-corrects for the larger pool, so wider promotion is safe

### Analysis Framework (10 sections)

1. **Landscape:** total configs, passing gates, passing per family
2. **Per-family top 5** by total_r (trades * expectancy). Show ALL: trigger params (extracted individually), exit_strategy, atr_sl, pct_sl, sl_type, entry_retrace_pct, rr_ratio, rr_ratio_2, trades, trades_per_month, total_months, gross/net_primary/net_conservative expectancy, win_rate, PF, max_drawdown, total_r, losing_months
3. **Total R vs expectancy:** compare top 5 by total_r vs top 5 by expectancy. Flag families where the rankings diverge -- high-volume moderate-edge configs that rank differently under each metric
4. **Trade volume / OOS feasibility:** for each family's best config, estimate OOS trades for WFT. Flag any below exploratory floor
5. **Exit robustness (policy-required):** for each family's top trigger config, count exit variants passing gates, show net_exp at rank 1/5/10/20. Label ROBUST (>100 passing, flat top), MODERATE (30-100), FRAGILE (<30 or sharp cliff)
6. **Trigger param diversity:** per family, how many unique trigger_params pass gates? Dominated by one or spread?
7. **Exit strategy patterns:** cross-tabulate exit_strategy x family, SL type distribution, entry retrace distribution, PCT vs ATR usage
8. **Losing months and drawdown:** top 5 per family with losing_months/total_months ratio. Correlation between expectancy and drawdown
9. **Fee sensitivity:** gross vs net_primary vs net_conservative, fee_drag %, flag families above 25% consumed
10. **Cross-family summary table** (sort by total_r descending):

| Family | Net Exp | Net Cons | Trades | TPM | Total R | WR | PF | DD | Lose Mo% | Exit Robust | Trig Diversity | Fee Drag% |

### Promotion Shortlist

Organize by tiers:

**Tier 1 -- highest conviction:** Best families. Multiple exit variants per trigger to test exit sensitivity. Multiple triggers to test trigger sensitivity.

**Tier 2 -- solid candidates:** Second-tier families with robust exits. 1-2 exits per trigger.

**Tier 3 -- stability/volume plays:** Lower expectancy but good stability or high total_r through volume.

**Tier 4 -- exploratory:** Lower conviction, included for diversity. 1 config each.

Per-config table:

| # | Exit | Net Exp | Trades | Total R | PF | DD | Why |
|---|------|---------|--------|---------|----|----|-----|

"Why" examples:
- "Peak expectancy and total_r"
- "Tests rr2 sensitivity (3.0 vs 2.8)"
- "Volume winner: 801 trades, near-tied on total_r"
- "Lowest DD in entire dataset"
- "Tests whether nt=5 carries real edge"

Always include:
- Exclusion rationale for families/configs NOT promoted
- Caveats for estimated values or near-boundary configs
- Total config count per tier

---

## Stage 4: Walk-Forward Test (WFT) Analysis

### Purpose

Main statistical and robustness validator before holdout. Answers: does the strategy generalize OOS across time? Is the winner credible after accounting for multiple testing?

### Structure

- Rolling windows: 18-month IS, 3-month OOS, 96-bar purge
- Regime tagging
- Holdout exclusion

### Policy Pass Conditions (ALL must be true)

1. retention >= 0.50
2. consistency >= 0.60
3. consistency statistically meaningful (binomial test)
4. aggregate OOS expectancy > 0
5. OOS PF >= 1.2 (< 500 trades), >= 1.15 (500-2000 trades), >= 1.1 (> 2000 trades)
6. drawdown within policy limit
7. aggregate OOS trades >= 100 hard floor, plus month-normalized exploratory and validated floors
8. coin breadth gate passes (>= 30% of coins or >= 5 coins)
9. expectancy CI lower bound > 0 (when sample sufficient)
10. DSR passes threshold (>0.95 high-conviction, >0.80 preliminary)
11. no regime failure where regime sample is sufficient

### Trade Count Floors (scale with OOS months)

- Hard floor: 100 aggregate OOS trades
- Exploratory floor: scaled from 200 trades / 48 OOS months reference
- Validated-promotion floor: scaled from 1000 trades / 48 OOS months reference
- Between exploratory and validated: label "underpowered / low-confidence"

### Analysis Framework (9 sections)

1. **Pass/fail summary:** per config, pass/fail on each of the 11 criteria. Count total passes
2. **OOS expectancy with CI:** per config, aggregate OOS expectancy, 95% CI bounds, CI lower bound > 0?
3. **Retention and consistency:** per config, retention rate, consistency rate, binomial p-value
4. **DSR (mandatory):** per config, DSR value, effective trial count used, threshold applied. Document how trial count was estimated
5. **Coin breadth:** per config, % of coins passing coin-level WFT checks (positive exp, retention > 0.50, drawdown within limit, sufficient trades). Flag configs where breadth is marginal
6. **Regime analysis (mandatory, per-window):** Read the per-window detail CSV (not just the summary). For each passing config, compute per-regime mean/median OOS expectancy and win rate (% of windows positive) across bear/bull/range. Flag any config where bear mean expectancy is near zero or negative -- these are bull-dependent and vulnerable to bear-heavy holdout periods. Flag configs with balanced cross-regime performance as more robust. Show regime distribution of the WFT period (% bull/bear/range days) for later comparison with holdout.
7. **Trade volume adequacy:** aggregate OOS trades vs exploratory and validated floors. Label each config as validated / exploratory / underpowered
8. **Regime-robustness labels:** For each passing config, assign: REGIME-ROBUST (positive mean exp in all 3 regimes, >= 50% windows positive in worst regime), BULL-DEPENDENT (bear mean exp < +0.05R or < 50% bear windows positive), BEAR-SPECIALIST (bear is strongest regime), or REGIME-MIXED (other patterns). This label carries forward to holdout interpretation.
9. **Cross-config comparison table:**

| Config | OOS Exp | Total R | Calmar | CI Lo | CI Hi | Retention | Consistency | PF | DD | DSR | Breadth | OOS Trades | Regime Label | Label | Pass? |

Sort this table by Calmar (total_r / max_dd) descending. This ranks configs by how much money they earn per unit of risk, accounting for trade volume. A high-volume config with moderate expectancy will rank above a low-volume config with high expectancy if it earns more total R per unit of drawdown.

### Promotion to Holdout

**IMPORTANT: Promote ALL WFT configs to holdout, not just pass-only.** Do NOT use --pass-only when running holdout. The WFT statistical gates (DSR, coin breadth) can filter out configs with real edges. WFT is one gate, holdout is the other -- configs must pass both to be promoted, but each gate is applied independently, not sequentially.

The WFT pass/fail label is still useful as a confidence indicator in the holdout analysis, but it should not prevent a config from being tested in holdout.

Present the WFT results in tiers for context:

**Tier 1 -- validated:** Pass all 11 criteria, above validated trade floor, strong DSR.
**Tier 2 -- exploratory / underpowered:** Pass most criteria but below validated trade floor.
**Tier 3 -- conditional / gate-failed:** Positive OOS expectancy but failed on DSR, coin breadth, or other statistical gates. Still promoted to holdout.

Per-config table:

| # | Config | OOS Exp | Total R | Calmar | CI Lo | DSR | Retention | Breadth | OOS Trades | Regime Label | Tier | Why |
|---|--------|---------|---------|--------|-------|-----|-----------|---------|------------|--------------|------|-----|

Always include:
- Which gate(s) each config failed and why
- Caveats for configs near boundaries (marginal DSR, borderline CI)
- Flag any config where regime sample was too sparse to test

---

## Stage 5: Holdout Analysis

### Purpose

Final historical validator. All design choices are already frozen. Answers: does the strategy work on data never used in discovery, tuning, or WFT?

### Current Status

Document whether holdout is pristine or research-consumed. If consumed, note that it cannot be treated as final evidence.

### Policy Rules

1. No tuning after review -- no changing thresholds, sweep logic, ranking metrics, or exit grids after reviewing holdout
2. If process is redesigned after holdout review, that holdout is consumed
3. Holdout is a validator, not a discovery stage -- do not rescue failing configs

### Regime Comparison (mandatory, run before interpreting results)

Before analyzing holdout performance, classify the holdout period's regime mix and compare it to the WFT period. Use BTC 1h data resampled to daily, with 200-day SMA and 14-day ADX (same logic as classify_oos_regime in walk_forward.py): bull = close > SMA200 AND ADX > 25, bear = close < SMA200 AND ADX > 25, range = ADX <= 25.

Compute:
- Holdout regime distribution: % of days in bull/bear/range
- WFT period regime distribution: % of days in bull/bear/range
- Monthly regime breakdown for the holdout period (dominant regime per month, BTC price range, ADX level)

Present the comparison table upfront. If the holdout is significantly more bear-heavy or bull-heavy than the WFT period, state this explicitly -- it directly affects interpretation of every config's holdout result.

Cross-reference with WFT regime labels from Stage 4: configs labeled BULL-DEPENDENT in WFT are expected to degrade in a bear-heavy holdout. Configs labeled REGIME-ROBUST should hold up. If BULL-DEPENDENT configs fail holdout but REGIME-ROBUST configs pass, that is regime mismatch, not strategy failure.

### Analysis Framework (7 sections)

1. **Regime comparison:** holdout vs WFT regime mix (as described above). State whether holdout is bull-heavy, bear-heavy, or balanced relative to WFT. This frames all subsequent interpretation.
2. **Expectancy and CI:** per config, holdout expectancy, 95% confidence interval. CI lower bound > 0?
3. **Trade count and volume:** total holdout trades, trades per month, sufficient sample?
4. **PF and drawdown:** profit factor, max drawdown during holdout period
5. **Coin breadth:** % of coins with acceptable holdout expectancy and adequate trade support
6. **Monthly/regime consistency:** monthly returns summary, any extended losing streaks? Performance in different regimes if tagged. Cross-reference with monthly regime breakdown from section 1 -- do losing months align with bear regime months?
7. **Holdout summary table:**

| Config | Holdout Exp | CI Lo | Trades | PF | DD | Breadth | Months | WFT Regime Label | Verdict |

### Verdict Categories

- **PASS:** positive expectancy, CI lower bound > 0, adequate trades, acceptable breadth and drawdown
- **MARGINAL:** positive expectancy but CI includes zero, or breadth marginal, or sample thin
- **FAIL:** negative expectancy, or CI clearly includes zero, or unacceptable drawdown/breadth
- **REGIME-EXPLAINED FAIL:** negative holdout expectancy BUT the config was labeled BULL-DEPENDENT in WFT AND the holdout was bear-heavy. Note this explicitly -- it is not the same as an unconditional failure. The trigger may still have edge in bull/range regimes.

### Promotion Shortlist to Paper Trading

**Dual-gate promotion: configs must be positive in BOTH WFT and holdout.** WFT provides the long-term expectancy estimate and regime profile. Holdout provides survival confirmation on recent unseen data. Neither alone is sufficient.

Selection criteria (in priority order):
1. Positive in both WFT and holdout (hard gate)
2. Regime balance in WFT -- prefer configs balanced across bull/bear/range over configs that peak in one regime
3. WFT expectancy for long-term projection (holdout is a survival check, not a ranking tool)
4. Trade volume, DD, monthly consistency as tie-breakers

Do NOT rank by holdout performance. A config with +0.05R holdout and +0.30R WFT (balanced regimes) beats a config with +0.25R holdout and +0.17R WFT. The holdout winner likely just suits the holdout's specific regime.

| # | Config | WFT Exp | HO Exp | Regime Label | WFT Trades | HO Trades | HO PF | HO DD | HO Months | Why |
|---|--------|---------|--------|--------------|------------|-----------|-------|-------|-----------|-----|

"Why" examples:
- "Positive both gates, balanced regimes, highest WFT exp in family"
- "Regime-robust, strong WFT consistency (0.87), holdout survival confirmed"
- "Best WFT+holdout balance (geometric mean), good trade volume"

Always include:
- Exclusion rationale for configs that failed either gate
- Whether holdout is pristine or consumed (affects interpretation weight)
- Any configs that passed WFT gates but failed holdout (regime mismatch learning)
- Any configs that failed WFT gates but passed holdout (gate too strict learning)
- For WFT passers that failed holdout: state their WFT regime label. If BULL-DEPENDENT configs failed a bear-heavy holdout, note this is regime mismatch, not unconditional failure.

### Post-Holdout

- Passing configs move to paper/shadow trading
- No further historical optimization after holdout review
- Future unseen months after policy freeze become the new lockbox
- REGIME-EXPLAINED FAIL configs should not be discarded entirely -- they may be valid for regime-conditional trading (e.g. run only when BTC > SMA200) or may recover when the holdout window shifts to include more bull exposure
