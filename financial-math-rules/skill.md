---
name: financial-math-rules
description: Accurate financial math rules. Use when code calculates money, prices, PnL, fees, or account balances.
---

# Financial Math Rules

## Overview

Financial calculations require exact arithmetic, correct fee handling, and exchange-compliant rounding. IEEE 754 floats cause drift; missing fees inflate PnL; wrong rounding causes order rejections.

## Rules

### 1. Use Decimal, Not Float, for Money

Python's `float` uses IEEE 754 binary representation. After 100+ operations, accumulated drift produces incorrect totals.

```python
from decimal import Decimal

# Correct
total_pnl = sum(Decimal(r["pnl"]) for r in rows)

# Wrong — drifts after many trades
total_pnl = sum(float(r["pnl"]) for r in rows)
```

Applies to: PnL accumulation, account value summation, fee totals, any running sum.

### 2. Always Deduct Both Entry and Exit Fees

Every trade has two fee events. Missing either inflates reported PnL.

```python
entry_fee = entry_price * size * TAKER_FEE_RATE   # 0.045%
exit_fee  = close_price * size * exit_fee_rate     # taker or maker

net_pnl = raw_pnl - entry_fee - exit_fee
```

Fee rate selection:
- Taker (0.045%): market orders, SL hits, emergency closes
- Maker (0.015%): limit fills, TP orders filled on book

Conservative default: assume taker on entry.

### 3. Close at Entry = Net Loss

A position closed at exactly the entry price loses two fees (entry + exit). This is the minimum cost of any round trip. Use as sanity check in tests.

### 4. Partial Close PnL Uses Partial Size

When closing part of a position (e.g., TP1 fills 50%), calculate PnL only for the closed portion:

```python
partial_pnl = (close_price - entry_price) * closed_size  # not full size
partial_pnl -= entry_price * closed_size * TAKER_FEE_RATE
partial_pnl -= close_price * closed_size * MAKER_FEE_RATE
```

### 5. Size Splits: Subtract, Don't Halve Twice

When splitting a position (e.g., 50/50 for TP1/TP2):

```python
tp1_size = round(size * 0.5, sz_decimals)
tp2_size = round(size - tp1_size, sz_decimals)  # NOT round(size * 0.5)
```

This guarantees tp1_size + tp2_size = size exactly.

### 6. Exchange Price Rounding (Hyperliquid)

Hyperliquid enforces:
- Max 5 significant figures
- Max (6 - szDecimals) decimal places
- Integers always allowed

After sig-fig rounding, re-check decimal limit (sig-fig rounding can violate it).

### 7. Order Size Tolerance Check

After rounding size to exchange precision, verify notional is within 1% of target:

```python
actual_usd = rounded_size * price
if abs(actual_usd - target_usd) > target_usd * 0.01:
    return None  # reject, too far from target
```

### 8. Account Value: Don't Double-Count

Exchange APIs may return overlapping balance fields. Only sum the specific field that represents actual value (e.g., USDC balance only, not total + breakdown).

### 9. Margin: Perps vs Spot

On unified accounts, spot holdings are not usable as perp margin. Always use perp-side margin for leverage and availability checks.

### 10. Blended Price for Multi-Exit

When a position closes in parts at different prices, use weighted average:

```python
blended = (size_a * price_a + size_b * price_b) / total_size
```

## Quick Reference

| Rule | Common Bug | Fix |
|------|-----------|-----|
| Decimal math | PnL drift after many trades | `Decimal` for sums |
| Both fees | PnL higher than actual | Deduct entry + exit |
| Partial PnL | Overstated partial close | Use closed_size, not full |
| Size split | Dust left behind | Subtract, don't halve twice |
| Price rounding | Order rejected | 5 sig figs, (6-szDec) decimals |
| Size tolerance | Unexpected position size | 1% check after rounding |
| Account value | Balance shows 2x actual | Don't sum overlapping fields |
| Margin source | "Insufficient margin" | Use perp margin, not spot |

## Testing

Use property-based testing (Hypothesis) to verify:
- Close at entry = negative PnL (two fees)
- Maker exit PnL > taker exit PnL (same trade)
- Partial PnL proportional to partial size
- Size splits sum to original size exactly
