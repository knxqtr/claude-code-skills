---
name: api-integration-patterns
description: Patterns for reliable external API integration — retries, caching, fuzzy matching, and error handling. Use when connecting to third-party APIs, exchanges, payment providers, or external services. Use when user says "API", "integration", "retry", "rate limit", "caching", or "external service".
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

## Common Mistakes

- Calling get_price() every second when a 2-second cache would cut API calls in half.
- Treating "order not found" on cancel as a crash-worthy error instead of a normal race condition.
- Hardcoding identifier mappings only for the assets you trade today. New assets will fail silently.
- Using production API keys during development.
