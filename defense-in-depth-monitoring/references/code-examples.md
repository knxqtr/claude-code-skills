# Defense in Depth Code Examples

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
Needed: 45 minutes  -> Use: 30 minutes (check 2x per period)
Needed: 2 hours     -> Use: 1 hour (check 2x per period)
Needed: 3 hours     -> Use: 1 hour (check 3x per period)
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

## Consecutive Error Escalation

When a polling loop fails repeatedly, escalate from logging to alerting after a threshold. Reset the counter on any success.

```python
_CONSECUTIVE_ERROR_ALERT_THRESHOLD = 10
consecutive_errors = 0

while running:
    try:
        result = await check()
        consecutive_errors = 0  # reset on success
    except Exception as e:
        consecutive_errors += 1
        logger.warning(f"Error #{consecutive_errors}: {e}")
        if consecutive_errors == _CONSECUTIVE_ERROR_ALERT_THRESHOLD:
            await alert(f"Loop failing repeatedly: {consecutive_errors} consecutive errors")
```

## Confirmation Before Acting on Disappearances

When something disappears from an external system (order gone, position gone), wait and re-check before acting. API glitches can return empty results temporarily.

```python
# BAD: position not found -> immediately declare it closed
# (could be a momentary API glitch)

# GOOD: wait 5 seconds, check again
if position_gone:
    await asyncio.sleep(5)
    position = await exchange.get_position(coin)
    if position is None:
        # confirmed gone, now act
        await handle_external_close(coin)
```
