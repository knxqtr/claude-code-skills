# Financial Math Rules -- Code Examples

## 1. Decimal, Not Float

```python
# BAD: 0.1 + 0.2 = 0.30000000000000004
total += 0.1 + 0.2

# GOOD: exact decimal arithmetic
from decimal import Decimal
total += Decimal("0.1") + Decimal("0.2")  # = Decimal("0.3")
```

When converting config values or other floats to Decimal, always wrap in str() first:

```python
# BAD: Decimal(0.00005) becomes Decimal('0.000049999999999999996...')
threshold = Decimal(config.MIN_THRESHOLD)

# GOOD: Decimal("0.00005") is exact
threshold = Decimal(str(config.MIN_THRESHOLD))
```

Use `//` (integer division) not `/` (float division) when computing integer
intermediate values that will be mixed with Decimals:

```python
# BAD: int / int produces float, float * Decimal raises TypeError
sum_x = n * (n - 1) / 2          # float!
result = sum_x * some_decimal     # TypeError

# GOOD: int // int produces int, int * Decimal works fine
sum_x = n * (n - 1) // 2         # int
result = Decimal(sum_x) * some_decimal  # Decimal
```

## 2. Use Actual Fill Prices

```python
# BAD: record intended price
trade.entry_price = signal.entry_price

# GOOD: query actual fill after placement
order = await exchange.place_order(...)
trade.entry_price = await exchange.get_fill_price(order.id)
```

The difference is slippage. Ignoring it means your PnL is wrong from the start.

## 3. Deduct Fees on Both Sides

```python
pnl = exit_value - entry_value
pnl -= entry_notional * taker_fee_rate   # entry fee
pnl -= exit_notional * maker_fee_rate    # exit fee (may differ)
```

For funding rate arb with two perp legs:

```python
# Entry: taker fees on both legs (conservative, assumes market orders)
entry_cost = taker_fee[long_exchange] + taker_fee[short_exchange]

# Exit: maker fees on both legs (planned exits use limit orders)
exit_cost = maker_fee[long_exchange] + maker_fee[short_exchange]

# Break-even: how many funding periods to recoup round-trip fees
total_round_trip = entry_cost + exit_cost
break_even_periods = total_round_trip / gross_spread_per_period
```

## 4. Know Which Balance You Are Reading

```python
# Common traps:
# - Total equity (includes spot + perps) vs available margin (perps only)
# - Account value vs withdrawable balance
# - Gross balance vs net-of-fees balance

# Always verify: "Does this number represent what I can actually use?"
```

## 5. Rounding Must Match the Target System

```python
# Each asset has its own precision requirement
# BTC: 4 decimal places for size, 1 for price
# ETH: 3 decimal places for size, 2 for price
# Check the API docs for each asset
```

## 6. Time Window Boundaries

```python
# Off-by-one errors cause double-counting or missed trades
# Use half-open intervals: >= start AND < end

start = datetime(2026, 3, 15, 0, 0, 0, tzinfo=UTC)
end = datetime(2026, 3, 16, 0, 0, 0, tzinfo=UTC)
trades = [t for t in all_trades if start <= t.timestamp < end]
```
