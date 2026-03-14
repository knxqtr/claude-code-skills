---
name: order-management-edge-cases
description: Order management edge cases. Use when building systems that place and manage orders — partial fills, cancel races, resizing.
---

# Order Management Edge Cases

## Overview

Order systems fail at boundaries: partial fills, cancel races, resize windows, and ambiguous disappearances. Every order operation must handle the case where the exchange state changed between your read and your write.

## Core Rules

### 1. Partial Fills Are Different From Full Fills

A partial fill creates a split state: active position + pending order remainder.

- Place TP immediately for the filled amount (don't wait for full fill)
- Start SL monitoring immediately
- Continue polling entry order for further fills
- Use 99% threshold: if filled >= target * 0.99, treat as full fill

### 2. Cancel-Then-Place, Never Place-Then-Cancel

Exchanges enforce: total reduce-only order size <= position size. You cannot place new TPs while old TPs exist.

Resize sequence:
1. Cancel old TP orders (individually, with rollback)
2. Re-check position size (TP may have filled during cancel)
3. Place new TP orders with updated size

### 3. Never Leave Partial Cancel State

If cancelling multiple orders and one fails:

```python
# Cancel TP1 — success
# Cancel TP2 — fails
# MUST re-place TP1 before aborting
if tp1_cancelled and not tp2_cancelled:
    re_place_tp1()  # restore protection
    return False     # resize failed, but position is protected
```

The position must always have protection. A half-cancelled state is worse than no resize.

### 4. Resize Windows Need Timestamps

Use a timestamp flag (not a boolean) when resize is in progress:

```python
trade.tp_resize_started_at = time.time()
```

Health monitors skip missing-order alerts if resize started < 30 seconds ago. Boolean flags can't express "resize started but might be stuck."

### 5. Verify Order Status, Don't Assume

When an order disappears, three things could have happened:
- Order filled (success)
- Order cancelled externally
- Position liquidated

Always call `was_order_filled(oid)` to distinguish. Never assume "order gone + position gone = TP filled."

### 6. Post-Fill Validation

Immediately after fill, check that SL and TP levels still make sense relative to actual entry price:

```python
if (is_long and entry >= tp) or (not is_long and entry <= tp):
    close_immediately()  # filled past TP, invalid state
```

Slippage can cause fills at prices where the trade setup is nonsensical.

### 7. One Pending Limit Per Coin

Track pending limits in a set. Reject duplicate signals for coins with active limit orders. Remove from set on: fill, cancel, TP reached, or order disappearance.

### 8. Undersized TP: Market Close Remainder

If a TP fills but position is still open (TP was sized smaller than position), market close the remainder immediately. Don't place a new limit order for the leftover — it leaves the position unprotected.

### 9. Atomic State Writes

Trade state persistence must be crash-safe:

```python
# Write to temp file, flush to disk, atomic rename
with open(tmp_path, "w") as f:
    json.dump(data, f)
    f.flush()
    os.fsync(f.fileno())
os.replace(tmp_path, final_path)  # atomic
```

### 10. Per-Coin Locking

Use per-coin async locks (not global) to serialize handlers. Two handlers for the same coin (e.g., TP health + position health) must not race.

## Recovery Cases

| Case | State | Action |
|------|-------|--------|
| A: JSON + Exchange | Full data | Restore SL/TP/strategy, restart monitors, check offline SL breaches |
| B: JSON + No Exchange | Closed offline | Clean up JSON, notify user |
| C: Exchange + No JSON | No saved state | Minimal recovery, match against recent signals for SL/TP |
| D: Neither | Nothing | No action |

## Quick Reference

| Edge Case | Wrong Approach | Correct Approach |
|-----------|---------------|-----------------|
| Partial fill | Wait for full fill | TP + SL immediately for filled portion |
| Cancel race | Ignore failed cancel | Rollback successful cancels, restore protection |
| Order disappeared | Assume TP filled | Call was_order_filled() API |
| Resize in progress | Boolean flag | Timestamp + 30s timeout window |
| Undersized TP fill | Place new limit for remainder | Market close remainder now |
| State persistence | Direct file write | Temp + fsync + atomic rename |
| Concurrent handlers | Global lock | Per-coin async lock |
| Post-fill price | Trust signal levels | Validate entry vs SL/TP, close if invalid |

## Common Mistakes

1. Treating partial fills as full fills (wrong TP sizing, delayed monitoring)
2. Placing new TPs before cancelling old ones (exchange rejects: reduce-only exceeded)
3. Leaving position unprotected after one cancel succeeds and another fails
4. Using boolean flags for resize state (can't detect stale resize)
5. Assuming order disappearance means it filled (could be liquidation)
6. Writing state directly to file (corruption on crash)
