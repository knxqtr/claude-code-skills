---
name: portfolio-monte-carlo-design
description: Correctness rules for multi-strategy portfolio Monte Carlo simulators that sample from empirical trade pools and compound a shared equity bucket. Use when building or reviewing a portfolio MC engine, debating per-trade vs periodic compounding, or debugging order-sensitive results. Use when user says "monte carlo", "compounding", "empirical bootstrap", "portfolio simulator", "order-invariant", or "median looks too high".
---

# Portfolio Monte Carlo Design

## When to Use

- Building a new Monte Carlo simulator that runs multiple strategies against a shared equity pool
- Reviewing an existing simulator whose median looks suspiciously high
- Deciding between per-trade, monthly, weekly, or daily compounding
- Debugging why reordering the strategy list changes results

## Rules

### 1. Per-Trade Compounding Silently Assumes Instantaneous Settlement

A multi-strategy portfolio simulator that iterates `for strategy in strategies: for trade in trades: equity *= (1 + risk_pct * trade_R)` gives later trades the benefit of earlier trades' fully realized compounding. The math is commutative, so this is not an ordering bias — but it is a timing optimism. The simulator has implicitly assumed each trade closes before the next one opens.

In a live parallel bot, that is not how positions work. Average hold times are typically hours to days. When signal B fires while position A is still open, the bot cannot size B off equity that already includes A's final P&L, because A has not closed yet. Margin for A is locked, and B must be sized off available-free-equity, which is strictly less than the sim-reported equity.

Per-trade compounding is correct for single-strategy backtests. For multi-strategy portfolios in no-timestamp simulators, it is the aggressive upper bound, not the point estimate.

**The conservative bracket: snapshot equity at period start, size every trade in the period off that snapshot.**

```python
# Aggressive bound: per-trade compounding (timing-optimistic)
for month in range(months):
    for strategy in strategies:
        for trade_R in sample_trades(strategy, rng):
            equity *= (1 + risk_pct * trade_R)

# Conservative bound: periodic snapshot sizing
for month in range(months):
    month_start_equity = equity.copy()
    for strategy in strategies:
        for trade_R in sample_trades(strategy, rng):
            # Trade P&L still lands on running equity, but sizing is fixed.
            equity += month_start_equity * risk_pct * trade_R
            equity = np.maximum(equity, 0.0)
```

Pick the snapshot period from your data boundary. If your empirical CSV only has month labels, monthly compounding is defensible. If you try to partition a month's trades into weekly or daily buckets without timestamp data, you are fabricating within-month structure, which is synthetic noise, not more resolution.

**Report the answer as a bracket, not a point estimate.** The realistic median lives between the per-trade (aggressive) and periodic-snapshot (conservative) runs. Do not pick one and present it as the answer — both are bounds, and the real bot lives somewhere between.

### 2. Size-Sensitive Costs Must Use the Snapshot, Not Running Equity

If you adopt periodic snapshot sizing, every code path that computes trade size in dollars must use the snapshot, not the running equity:

- Notional calculation (for market impact / slippage): `notional = snapshot_equity * (risk_pct / stop_pct)`
- Fee dollars (for reporting): `fee_dollars = snapshot_equity * risk_pct * fee_r`
- Position size for exchange-tier lookups: same snapshot

Running equity still changes with every trade (wins add, losses subtract), but the sizing base stays fixed until the next period boundary. Missing any one of these paths produces a hybrid that is neither snapshot nor per-trade, and the number is hard to interpret.

### 3. Fixed-Size Sizing Raises Ruin Risk — That is Honest, Not a Bug

Per-trade compounding has a defensive ratchet: during a drawdown, the next trade is sized off shrunk equity, so the dollar risk per trade shrinks automatically. Periodic snapshot sizing does not do this within the period. If equity drops 30% mid-month, every remaining trade that month still risks `start-of-month * risk_pct` in dollars — a much larger fraction of current equity.

Expect ruin rates under snapshot sizing to be noticeably higher than under per-trade sizing at the same nominal risk level. This is not a flaw in the conservative bracket. It is a truthful exposure of what fixed-dollar sizing actually costs during drawdowns. The correct response is a kill-switch in the live trading plan (halt trading at an equity drawdown threshold), not switching compounding modes to hide the risk.

### 4. Derive One RNG Per Sampling Unit, Never Share a Global Stream

Sharing a single `numpy.random.Generator` across multiple sampling units (strategies, sims, tests) makes results sensitive to iteration order in a way that looks like an engine bug. Reordering strategies in a YAML list, or swapping two tests, causes each unit to draw from a different slice of the shared stream.

This is not a commutativity problem in the math. It is a state-sharing problem in the RNG. Fix it at construction:

```python
import hashlib
import numpy as np

def derive_rng(master_seed: int, unit_name: str) -> np.random.Generator:
    # SHA-256 of a prefixed key makes the derivation:
    #   (a) deterministic across runs,
    #   (b) independent of iteration order,
    #   (c) collision-resistant across units.
    digest = hashlib.sha256(f"mc:{master_seed}:{unit_name}".encode("utf-8")).digest()
    sub_seed = int.from_bytes(digest[:8], "little", signed=False)
    return np.random.default_rng(sub_seed)

master_rng = np.random.default_rng(config.seed)  # for any order-independent global draws
unit_rngs = {name: derive_rng(config.seed, name) for name in unit_names}
```

Inside the loop, every per-unit sampling call uses `unit_rngs[name]`, never the master rng. The master rng is reserved for draws that live outside the per-unit loop (e.g. a month-level joint sampler that applies to all units at once).

### 5. Assert Order Invariance, Do Not Just Believe It

Write a regression test that runs the same simulation with units in two different orders and asserts `np.allclose` on key outputs (`final_values`, `max_drawdowns`, aggregated R totals). Do not use byte-for-byte hash equality — multiplication is associative mathematically but not in IEEE 754 floating point, so different orderings will differ by under one ULP of rounding noise even when the draws are identical.

```python
def test_engine_is_invariant_to_unit_list_order():
    forward = run_simulation(units=[a, b], config=cfg)
    reversed_ = run_simulation(units=[b, a], config=cfg)
    assert np.allclose(forward.final_values, reversed_.final_values, rtol=1e-9, atol=1e-6)
    assert np.allclose(forward.max_drawdowns, reversed_.max_drawdowns, rtol=1e-9, atol=1e-9)
```

If this test ever produces differences outside the tolerance, the RNG derivation has regressed — a new sampling site was added that still uses the shared stream.

## Common Mistakes

- Running per-trade compounding for a multi-strategy portfolio and quoting the median as "the" number. Report the bracket.
- Partitioning a month's trades into sub-month buckets without real timestamps. You are fabricating data structure that does not exist in the source.
- Fixing the sizing path but forgetting to fix the notional/fee-dollar calculations. Produces a hybrid that is neither snapshot nor per-trade.
- Switching from snapshot compounding back to per-trade because the ruin number looks scarier. The ruin is truthful; the right fix is a kill-switch, not hiding it.
- Using `assert_array_equal` on order-invariance tests. Multiplication reorderings differ by under one ULP; use `np.allclose`.
- Sharing one `numpy.random.Generator` across all sampling units. Any reordering of units produces different draws, and the diffs look like real engine changes.
- Deriving per-unit rngs via `SeedSequence.spawn(n)`. `spawn()` depends on the current SeedSequence state, so the result still depends on call order. Use a hash of a stable unit identifier instead.
