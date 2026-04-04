---
name: backtesting-validation
description: Simulation correctness rules for walk-forward testing (WFT) frameworks and holdout validation pipelines. Use when building or debugging WFT, holdout tests, or any multi-stage backtest pipeline. Covers window isolation, engine consistency, and pipeline data flow. Use when user says "walk-forward", "holdout", "OOS", "out-of-sample", "cross-window", or "backtest stages".
---

# Backtesting Validation Rules

## When to Use

- Building or extending a walk-forward testing framework
- Writing holdout / out-of-sample validation scripts
- Debugging cases where WFT passes but holdout fails (or vice versa)
- Connecting multiple pipeline stages (sweep -> WFT -> holdout)

## Rules

### 1. Simulate Per OOS Window, Never Global-Simulate-Then-Filter

Walk-forward testing requires running the simulation separately for each OOS window slice using only that window's data, not running one global simulation across all data and then filtering results by date.

**Wrong:**
```python
# All trades across full history
all_trades = simulate(data=full_history)
# Then filter to OOS window -- BROKEN: overlap prevention state bleeds between windows
oos_trades = [t for t in all_trades if window.start <= t.timestamp < window.end]
```

**Right:**
```python
for window in walk_forward_windows:
    # Each window gets only its own OOS data with fresh simulation state
    trades = simulate(data=window.oos_slice, fresh_state=True)
    window.record(trades)
```

Why this matters: Simulations with overlap prevention (one trade per trigger per coin, no opposing-direction trades) maintain global state. Running once across all data means window N's open trades block window N+1's entries. The resulting OOS metrics are not independent.

### 2. Holdout Must Use the Exact Same Simulation Engine as Training Stages

The holdout validation script must call the same simulation function as the parameter sweep and WFT. Using a simplified engine "for speed" silently ignores features that the training stages tested.

**Checklist before running holdout:**
- [ ] Same entry mode (market vs limit/retrace) as sweep configs
- [ ] Same SL type (candle close vs wick) as sweep configs
- [ ] Same exit strategy (Strategy A/B/D) as sweep configs
- [ ] Same filter parameter forwarding as sweep configs

A config that uses limit entry + close SL + Strategy D in training will produce meaningless holdout results if holdout uses market entry + wick SL + Strategy A. The holdout numbers will differ from training not because of regime change but because the simulation is wrong.

**Detection:** If every config in a holdout batch produces identical results regardless of their TP2 values, the simulation is probably ignoring multi-TP logic. Run a sanity check with configs that have meaningfully different TP2 settings and verify the outputs differ.

### 3. Save All Pipeline Stage Outputs to Files Before Running Downstream Stages

Every stage (component sweep, parameter sweep, exit sweep, WFT, holdout) must write its results to a file before the next stage runs. Never depend on terminal output or in-memory data across stages.

**Wrong:**
```bash
python sweep.py | grep "top configs"  # Results lost after terminal closes
python walk_forward.py               # Must find configs from... where?
```

**Right:**
```bash
python sweep.py --output sweep_results_v1.csv
# Verify file exists before proceeding
python walk_forward.py --input sweep_results_v1.csv --output wft_results_v1.csv
```

Check that the upstream file exists and has the expected row count before launching a downstream stage. Long sweeps (hours or days) are wasted if the output file was not written.

### 4. Training and Holdout Data Must Not Overlap

The holdout period must be strictly reserved — no part of it can appear in the training or WFT windows. Confirm the cutoff date by checking the earliest timestamp in the holdout data against the latest timestamp in the training data. A single overlapping bar invalidates the holdout result.

Store the cutoff date as a constant in the codebase (not a CLI default) to prevent accidental changes between runs.

## Common Mistakes

- Running `simulate(full_history).filter(by=oos_window)` instead of `simulate(oos_slice_only)`. Looks correct; produces subtly wrong metrics when overlap-prevention state crosses window boundaries.
- Using a simplified "speed" engine for holdout that ignores limit entry, multi-TP exits, or close SL. Holdout numbers become incomparable with training.
- Not verifying the upstream CSV exists before starting a downstream sweep. Hours wasted when the file path was wrong.
- Hardcoding the holdout cutoff date as a CLI default argument. It gets accidentally changed when the user experiments with different flags.
- Forgetting that configs with different TP2 values should produce different holdout results. If they don't, the multi-TP engine is not being called.
