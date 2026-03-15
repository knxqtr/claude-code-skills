---
name: financial-math-rules
description: Accurate financial math rules. Use when code calculates money, prices, PnL, fees, or account balances.
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
5. **Rounding Must Match the Target System** -- Each asset has its own decimal precision requirement. Rounding incorrectly causes order rejections.
6. **Time Window Boundaries** -- Use half-open intervals (>= start AND < end). Off-by-one errors cause double-counting or missed trades.

For code examples, see references/code-examples.md in this skill's directory.

## Common Mistakes

- Using float for a running PnL total that accumulates over hundreds of trades.
- Recording the intended order price instead of querying the actual fill price.
- Forgetting fees exist, or only deducting on one side.
- Using spot balance for perp margin calculations (or vice versa).
- Hardcoding rounding to 2 decimal places when the asset requires 4 or 8.
- Using `Decimal(0.00005)` instead of `Decimal(str(0.00005))` -- IEEE 754 contaminates the Decimal.
- Using `int / int` in a formula with Decimals -- produces float, causes TypeError.
