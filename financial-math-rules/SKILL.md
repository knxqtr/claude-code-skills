---
name: financial-math-rules
description: Accurate financial math rules for trading systems, payment processing, and accounting. Use when code calculates money, prices, PnL, fees, or account balances. Covers Decimal vs float, actual fill prices, fee deduction, balance types, rounding precision, and time window boundaries. Use when user says "rounding error", "fee calculation", or "balance mismatch".
---

# Financial Math Rules

## When to Use

- Any code that calculates money, prices, or percentages
- Trading systems, payment processing, invoicing, accounting
- Displaying account balances or transaction history
- Any arithmetic where small errors accumulate over time

## Rules

1. **Decimal, Not Float** -- IEEE 754 floats accumulate rounding errors. Use Decimal for all PnL, fee, and balance calculations. When converting floats to Decimal, wrap in str() first: `Decimal(str(value))`. Use `//` not `/` for integer intermediates mixed with Decimal.
2. **Use Actual Fill Prices** -- Never record the intended price. Query the exchange for the actual fill price after placement. The difference is slippage.
3. **Deduct Fees on Both Sides** -- Every trade has fees on entry AND exit. Taker and maker rates are usually different. Omitting fees makes reported PnL higher than actual.
4. **Know Which Balance You Are Reading** -- Total equity vs available margin vs withdrawable balance. Using the wrong one causes sizing errors and false safety checks.
5. **Rounding Must Match the Target System** -- Each asset has its own decimal precision requirement. Rounding incorrectly causes order rejections. When porting indicator math between platforms (Python to Pine Script, Python to exchange API), verify the rounding/truncation method. Python's `round()` uses banker's rounding (round half to even); Pine Script and most exchanges truncate with `int()`. For `round(3.5)` Python returns 4, `int(3.5)` returns 3.
6. **Time Window Boundaries** -- Use half-open intervals (>= start AND < end). Off-by-one errors cause double-counting or missed trades.
7. **Unit Consistency in Comparisons** -- When comparing two financial values, verify they share the same units. Dollar PnL (price_delta * size) and price distance (atr * multiplier) are both floats but have different units. Comparing them produces silently wrong results. Any comparison: check left-side units == right-side units.
8. **Defer Price-Relative Fields Until Fill** -- In order execution systems (backtester or live), do not set stop-loss, take-profit, or other price-relative fields until the actual fill price is known. The intended entry price and actual fill price can differ (slippage, next-bar fills). Set these fields in a post-fill callback, not at signal generation time.
9. **Lock/Unlock Must Balance** -- When value is locked (margin, escrow, holds, reserves), the lock must be a debit from available balance. If you only credit on unlock without debiting on lock, you create phantom money. Always trace a full round-trip: debit at lock, credit at unlock. Test with a zero-profit round-trip to verify the balance returns to its starting value.
10. **Leverage Means Fraction-of-Margin** -- In leveraged trading, "fraction" conventionally means fraction of capital posted as margin (collateral). Leverage amplifies that into notional exposure. 20% fraction at 5x = 20% margin controlling 100% notional. If a sizer function accepts a leverage parameter but ignores it, the sizing model silently differs from what traders expect.
11. **Guard Every Division in Indicators** -- Financial indicator formulas frequently divide by values that can be zero in real data: flat prices (high == low), zero volume, constant RSI, zero ATR. Every division needs a guard (replace zero denominator with NaN, or return a sensible default like 50 for oscillators). This is not theoretical -- it happens regularly with low-liquidity crypto pairs.

12. **Separate NaN Sources in Indicators** -- Indicator series can have NaN from multiple causes: warmup period (rolling window not filled), division by zero (flat range), and missing input data. Each cause requires a different fix (warmup: leave as NaN, div-by-zero: return neutral default like 50 for oscillators, missing: forward-fill or skip). Using a blanket `fillna()` to fix one cause silently corrupts the others. Save the warmup mask before fixing div-by-zero, then restore it after.
13. **Mirror Short Exits Correctly for Bounded Oscillators** -- For oscillators with fixed bounds (RSI 0-100, WillR -100 to 0, MFI 0-100), long exits at the upper bound (e.g., RSI > 70) mirror to short exits at the lower bound (e.g., RSI < 30). The direction of the comparison operator flips AND the threshold moves to the opposite end. A common mistake is flipping only the threshold but not the operator direction, producing conditions that are true most of the time.
14. **Include All Costs in Per-Trade PnL** -- trade.pnl should reflect the complete cost of the round trip (entry commission + exit commission + any holding costs). If entry cost is deducted from capital separately at entry time, the capital restoration at exit must add it back to avoid double-counting. Incomplete per-trade PnL distorts profit_factor, avg_winner, avg_loser, and any downstream metric derived from individual trade results.

For code examples, see references/code-examples.md in this skill's directory.

## Common Mistakes

- Using float for a running PnL total that accumulates over hundreds of trades.
- Recording the intended order price instead of querying the actual fill price.
- Forgetting fees exist, or only deducting on one side.
- Using spot balance for perp margin calculations (or vice versa).
- Hardcoding rounding to 2 decimal places when the asset requires 4 or 8.
- Using `Decimal(0.00005)` instead of `Decimal(str(0.00005))` -- IEEE 754 contaminates the Decimal.
- Using `int / int` in a formula with Decimals -- produces float, causes TypeError.
- Comparing dollar PnL against a price distance without realizing they have different units (off by position size factor).
- Setting stop-loss on a Signal before the fill price is known -- the stop distance will be wrong if the fill price differs from the signal price.
- Locking margin/escrow without debiting available balance. The unlock credits it back, creating money from nothing. Every lock needs a matching debit.
- Accepting a leverage parameter in a sizing function but not using it. Traders expect leverage to amplify exposure; ignoring it produces undersized positions.
- Leaving indicator divisions unguarded because "zero prices don't happen." They do -- flat candles, zero volume, and constant oscillator values are common in crypto.
- Using `fillna(neutral_value)` to fix division-by-zero NaN, which also fills warmup NaN with fake data. Save the warmup mask first.
- Mirroring long exit `> 70` to short exit `> 30` instead of `< 30`. The operator direction flips along with the threshold.
- Reporting per-trade PnL that excludes entry commission. Capital tracking may be correct while per-trade metrics are silently optimistic.
