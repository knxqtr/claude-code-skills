---
name: api-integration-patterns
description: Reliable API integration patterns including retries, caching, rate limits, and identifier mapping. Use when connecting to external APIs, handling 429 errors, reducing redundant calls, or mapping asset/entity names between systems. Not for internal function calls.
---

# API Integration Patterns

## When to Use

- Integrating with any external API (exchange, payment, SaaS)
- Handling unreliable or slow API responses
- Mapping identifiers between systems
- Reducing unnecessary API calls

## Patterns

### 1. Price/Data Caching with Short TTL

Avoid hammering the API for data that does not change every millisecond.

```python
_cache = {}
_cache_time = {}
CACHE_TTL = 2  # seconds

async def get_price(coin):
    now = time.time()
    if coin in _cache and (now - _cache_time[coin]) < CACHE_TTL:
        return _cache[coin]
    price = await api.fetch_price(coin)
    _cache[coin] = price
    _cache_time[coin] = now
    return price
```

### 2. Skip Redundant Calls

If the API state matches what you want, do not call again.

```python
_leverage_cache = {}

async def set_leverage(coin, target):
    if _leverage_cache.get(coin) == target:
        return  # already set
    await api.set_leverage(coin, target)
    _leverage_cache[coin] = target
```

### 3. Fuzzy Identifier Matching

Different systems use different names for the same thing. Build a mapper.

```python
# Signal says "ASTR/USDT", exchange wants "ASTR"
# Signal says "BTC/USDT", exchange wants "BTC"
# Some names don't match at all: "SHIB/USDT" → "SHIB1000" on some exchanges

KNOWN_MAPPINGS = {"SHIB": "SHIB1000", "LUNA": "LUNA2", ...}

def normalize_coin(raw_pair):
    coin = raw_pair.split("/")[0].upper()
    return KNOWN_MAPPINGS.get(coin, coin)
```

### 4. "Not Found" on Cancel Means Already Filled

When cancelling an order that filled between your check and your cancel, the API returns "order not found." Treat this as success, not an error.

```python
try:
    await exchange.cancel_order(order_id)
except OrderNotFoundError:
    pass  # already filled or cancelled — this is fine
```

### 5. Testnet First

Always develop and test against the sandbox/testnet environment before touching production. Archive testnet data separately before migrating.

### 6. Retry with Context

Do not blindly retry. Log what failed and why, and limit retries.

```python
async def _retry(fn, *args, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            return await fn(*args, **kwargs)
        except TransientError as e:
            logger.warning(f"Attempt {attempt+1} failed: {e}")
            if attempt < max_retries - 1:
                await asyncio.sleep(2 ** attempt)
    raise RetryExhaustedError(f"Failed after {max_retries} attempts")
```

### 7. Use SDK Exceptions, Not Raw HTTP Errors

When an SDK provides its own exception hierarchy, catch those instead of raw HTTPError. SDK exceptions carry structured data (status codes, error messages) and survive SDK upgrades better.

```python
# Bad: catches raw HTTP, breaks if SDK changes transport
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 429: ...

# Good: catches SDK-provided exceptions
from sdk.errors import ClientError, ServerError

except ClientError as e:
    if e.status_code == 429:  # rate limit — retryable with backoff
        delay = base_delay * (2 ** attempt)
        await asyncio.sleep(delay)
    else:  # other 4xx — not retryable
        raise
except ServerError:  # 5xx — transient, retry
    await asyncio.sleep(base_delay)
```

### 8. Track API Weight, Not Call Count

Many APIs (Hyperliquid, Binance, etc.) rate-limit on **weight**, not raw call count. Different endpoints cost different amounts. Track weight per minute, not calls per minute.

```python
# Bad: count calls — 100 lightweight calls and 100 candle calls look the same
_call_count += 1

# Good: track weight — candle calls cost 10x more than price checks
_call_log.append((time.time(), weight))

def get_weight_per_minute(window_minutes=1):
    cutoff = time.time() - (window_minutes * 60)
    return sum(w for t, w in _call_log if t >= cutoff) / window_minutes
```

### 9. Audit ALL Polling Loops for Rate Limit Budget

When calculating API budget, account for every background loop — not just the obvious ones. Hidden consumers (pending order fill checkers, health monitors, reconnection handlers) can double the actual usage.

```
# Example: missed budget item
# Visible: 6 position monitors × 30s candle poll = known cost
# Hidden: 2 pending limit monitors × 2s poll × 2 API calls each = 240 extra weight/min
# Result: 429 rate limit errors at night when new positions opened
```

Rule: list every `asyncio.sleep` loop that makes API calls, multiply out the weight, and sum them all.

## Common Mistakes

- Calling get_price() every second when a 2-second cache would cut API calls in half.
- Treating "order not found" on cancel as a crash-worthy error instead of a normal race condition.
- Hardcoding identifier mappings only for the assets you trade today. New assets will fail silently.
- Using production API keys during development.
- Tracking API calls instead of API weight. A system can look healthy at 150 calls/min but be over budget at 1100 weight/min because of heavy endpoints.
- Forgetting background polling loops when calculating rate limit budget. If you only audit the main monitoring loops, hidden pollers (fill checkers, health checks) will push you over the limit at scale.
