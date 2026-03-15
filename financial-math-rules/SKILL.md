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

### 1. Decimal, Not Float

IEEE 754 floats accumulate rounding errors. Over hundreds of operations, cents become dollars.

```python
# BAD: 0.1 + 0.2 = 0.30000000000000004
total += 0.1 + 0.2

# GOOD: exact decimal arithmetic
from decimal import Decimal
total += Decimal("0.1") + Decimal("0.2")  # = Decimal("0.3")
```

Use Decimal for all PnL, fee, and balance calculations. Use float only for non-financial values like percentages used in display.

When converting config values or other floats to Decimal, always wrap in str() first:

```python
# BAD: Decimal(0.00005) becomes Decimal('0.000049999999999999996...')
threshold = Decimal(config.MIN_THRESHOLD)

# GOOD: Decimal("0.00005") is exact
threshold = Decimal(str(config.MIN_THRESHOLD))
```

Also use `//` (integer division) not `/` (float division) when computing integer intermediate values that will be mixed with Decimals. `int / int` produces a float in Python 3, and `float * Decimal` raises TypeError.

### 2. Use Actual Fill Prices

Never use the intended price as the entry price. Always query the exchange/system for the actual fill price after order placement.

```python
# BAD: record intended price
trade.entry_price = signal.entry_price

# GOOD: query actual fill after placement
order = await exchange.place_order(...)
trade.entry_price = await exchange.get_fill_price(order.id)
```

The difference is slippage. Ignoring it means your PnL is wrong from the start.

### 3. Deduct Fees on Both Sides

Every trade has fees on entry AND exit. Omitting them makes reported PnL higher than actual.

```python
pnl = exit_value - entry_value
pnl -= entry_notional * taker_fee_rate   # entry fee
pnl -= exit_notional * maker_fee_rate    # exit fee (may differ)
```

Know your fee rates. Taker and maker rates are usually different.

### 4. Know Which Balance You Are Reading

External systems often expose multiple balance figures. Using the wrong one causes sizing errors and false safety checks.

Common traps:
- Total equity (includes spot + perps) vs available margin (perps only)
- Account value vs withdrawable balance
- Gross balance vs net-of-fees balance

Always verify: "Does this number represent what I can actually use for this operation?"

### 5. Rounding Must Match the Target System

Exchanges and payment systems have specific decimal place requirements per asset. Rounding incorrectly causes order rejections.

```python
# Each asset has its own precision requirement
# BTC: 4 decimal places for size, 1 for price
# ETH: 3 decimal places for size, 2 for price
# Check the API docs for each asset
```

### 6. Time Window Boundaries

Off-by-one errors in time-based financial reports (daily PnL, weekly summaries) cause trades to be double-counted or missed entirely.

- Define clear boundaries: daily = midnight UTC to midnight UTC
- Use >= start AND < end (half-open intervals)
- Test around boundaries explicitly

## Common Mistakes

- Using float for a running PnL total that accumulates over hundreds of trades.
- Recording the intended order price instead of querying the actual fill price.
- Forgetting fees exist, or only deducting on one side.
- Using spot balance for perp margin calculations (or vice versa).
- Hardcoding rounding to 2 decimal places when the asset requires 4 or 8.

For detailed code examples, see `references/code-examples.md`.
