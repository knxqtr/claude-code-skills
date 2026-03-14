---
name: order-management-edge-cases
description: Edge cases in order management — partial fills, cancel races, resize windows, and size calculations. Use when building trading systems, order books, or any system that places and manages orders on an exchange. Use when user says "order", "partial fill", "cancel order", "resize", "take profit", or "order management".
---

# Order Management Edge Cases

## When to Use

- Building any system that places orders on an exchange
- Managing take-profit, stop-loss, or other conditional orders
- Handling order lifecycle events (fill, partial fill, cancel, expire)

## Edge Cases

### 1. Partial Fills Are Not Full Fills

A partial fill means some of the order executed but not all. Your code must reduce the position/order, not remove it.

```python
# BAD: any fill removes the entire position
if order_filled:
    del positions[coin]

# GOOD: reduce by filled amount
if order_filled:
    positions[coin].size -= filled_size
    if positions[coin].size <= 0:
        del positions[coin]
```

### 2. Cancel Racing with Fill

If you cancel an order that filled between your check and your cancel, the API returns "not found." This is normal, not an error.

```python
try:
    await exchange.cancel_order(order_id)
except OrderNotFoundError:
    # Order already filled or cancelled. Check if it filled.
    if await exchange.was_order_filled(order_id):
        await handle_fill(order_id)
```

### 3. Order Size Depends on Context

When placing a take-profit order, the size depends on whether there are multiple TPs:

```python
# Single TP: close 100% of position
tp_size = position.size

# Two TPs: TP1 closes 50%, TP2 closes remaining 50%
tp1_size = position.size / 2
tp2_size = position.size - tp1_size  # use subtraction, not division again
```

### 4. Mid-Resize Protection Window

When resizing an order (cancel old, place new), there is a window where no order exists. Health checks running during this window may incorrectly detect the order as missing.

```python
# Set a flag with a timeout before resizing
trade.resize_started_at = time.time()

# In health check loop:
if tp_order_missing:
    if trade.resize_started_at and (time.time() - trade.resize_started_at) < 30:
        return  # mid-resize, skip this check
    await alert_tp_missing()
```

### 5. Resize Error Recovery

Cancel-then-place is a two-step operation. Either step can fail independently:

- Cancel succeeds, place fails: order is gone with no replacement. Immediately fall back to a simpler order (e.g., single TP covering full position).
- Cancel fails: old order still active. Do not place a second order (would double up).
- Crash mid-resize: on recovery, detect missing orders and re-place.

### 6. Fill Notification Deduplication

Order fills can be detected by multiple code paths (polling, websocket, health check). Track whether you have already notified for a fill.

```python
if order_filled and not trade.notified_filled:
    await notify_fill(trade)
    trade.notified_filled = True
```

### 7. Verify Fill Source

"Order gone + position gone" does not always mean the order filled. It could be a liquidation, manual close, or external action. Verify with the exchange whether the specific order actually executed.

## Common Mistakes

- Treating partial fills as full fills. Position tracking goes wrong and subsequent orders have incorrect sizes.
- Crashing on "order not found" during cancel instead of treating it as a normal race condition.
- Placing TP1 for 100% of position when TP2 exists (should be 50%).
- Health check alerting "TP missing" during a resize window.
- Reporting a liquidation as a TP hit because both result in order + position disappearing.
