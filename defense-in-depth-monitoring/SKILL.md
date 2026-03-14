---
name: defense-in-depth-monitoring
description: Multi-layer event detection for reliable monitoring. Use when building health checks, alerts, or any system where missing an event has consequences.
---

# Defense in Depth Monitoring

## When to Use

- Monitoring for time-sensitive events (price triggers, SLA breaches, health checks)
- Any system where missing an event has real consequences
- Building alerting or notification systems
- Detecting state changes in external systems

## The Three-Layer Detection Pattern

No single detection method is 100% reliable. Use three layers:

### Layer 1: Precise Timer (lowest latency)
Calculate the exact time the event should occur. Sleep until that time, then check.
- Best latency (seconds)
- Fragile: depends on knowing the exact schedule
- Add a small buffer (2-3 seconds) to account for delays

### Layer 2: Regular Polling (medium latency)
Check at fixed intervals (15-60 seconds) regardless of the timer.
- Catches events the timer missed
- Works for non-standard schedules
- 0 to interval_length latency

### Layer 3: Safety Net (highest latency)
If you expected data but did not get it, fall back to checking raw state.
- Catches cases where the data source is delayed
- Require multiple consecutive confirmations (2+ checks 15 seconds apart)
- Guards against stale or glitchy data

When Layer 1 fires, cancel the next Layer 2 check to avoid duplicate processing.

## Time Guards on Historical Data

When checking historical data after a restart or reconnection, only consider events that happened AFTER the relevant start time.

```python
# BAD: check all historical candles (includes ones before the trade opened)
for candle in candles:
    if candle.close > stop_loss:
        trigger_stop()  # FALSE TRIGGER on pre-trade data

# GOOD: filter by trade open time
for candle in candles:
    if candle.timestamp_ms < trade.opened_at_ms:
        continue  # skip pre-trade candles
    if candle.close > stop_loss:
        trigger_stop()
```

## Non-Standard Interval Mapping

If your data source does not support the exact interval you need, map to the nearest smaller supported interval and check more frequently.

```
Needed: 45 minutes  → Use: 30 minutes (check 2x per period)
Needed: 2 hours     → Use: 1 hour (check 2x per period)
Needed: 3 hours     → Use: 1 hour (check 3x per period)
```

## Alert Deduplication

Health checks that run on a loop must track whether they have already alerted for the current issue. Without this, the user gets the same alert every cycle.

```python
# BAD: alert every 60 seconds while the issue persists
if tp_order_missing:
    await notify("TP order missing!")  # fires every minute

# GOOD: alert once, set flag
if tp_order_missing and not self._tp_missing_alerted:
    await notify("TP order missing!")
    self._tp_missing_alerted = True
```

Reset the flag when the issue resolves.

## Confirmation Before Acting on Disappearances

When something disappears from an external system (order gone, position gone), wait and re-check before acting. API glitches can return empty results temporarily.

```python
# BAD: position not found → immediately declare it closed
# (could be a momentary API glitch)

# GOOD: wait 5 seconds, check again
if position_gone:
    await asyncio.sleep(5)
    position = await exchange.get_position(coin)
    if position is None:
        # confirmed gone, now act
        await handle_external_close(coin)
```

## Common Mistakes

- Single-layer detection: one polling loop that can miss events if timing is unlucky.
- Checking historical data without filtering by start time. Causes false triggers on data that predates the monitored item.
- Alerting every poll cycle instead of once per issue. User gets 60 alerts per hour.
- Acting immediately on a disappearance without confirmation. API glitches cause false positives.
