# API Integration Patterns — Code Examples

## 1. Price/Data Caching with Short TTL

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

## 2. Skip Redundant Calls

```python
_leverage_cache = {}

async def set_leverage(coin, target):
    if _leverage_cache.get(coin) == target:
        return  # already set
    await api.set_leverage(coin, target)
    _leverage_cache[coin] = target
```

## 3. Fuzzy Identifier Matching

```python
# Signal says "ASTR/USDT", exchange wants "ASTR"
# Some names don't match: "SHIB/USDT" → "SHIB1000" on some exchanges

KNOWN_MAPPINGS = {"SHIB": "SHIB1000", "LUNA": "LUNA2", ...}

def normalize_coin(raw_pair):
    coin = raw_pair.split("/")[0].upper()
    return KNOWN_MAPPINGS.get(coin, coin)
```

## 4. "Not Found" on Cancel Means Already Filled

```python
try:
    await exchange.cancel_order(order_id)
except OrderNotFoundError:
    pass  # already filled or cancelled — this is fine
```

## 6. Retry with Context

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

## 7. Use SDK Exceptions, Not Raw HTTP Errors

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

## 8. Track API Weight, Not Call Count

```python
# Bad: count calls — 100 lightweight calls and 100 candle calls look the same
_call_count += 1

# Good: track weight — candle calls cost 10x more than price checks
_call_log.append((time.time(), weight))

def get_weight_per_minute(window_minutes=1):
    cutoff = time.time() - (window_minutes * 60)
    return sum(w for t, w in _call_log if t >= cutoff) / window_minutes
```

## 9. Audit ALL Polling Loops

```
# Example: missed budget item
# Visible: 6 position monitors × 30s candle poll = known cost
# Hidden: 2 pending limit monitors × 2s poll × 2 API calls each = 240 extra weight/min
# Result: 429 rate limit errors at night when new positions opened
```

Rule: list every `asyncio.sleep` loop that makes API calls, multiply out the weight, and sum them all.

## 10. Same Data, Different Field Names

```python
# Hyperliquid predictedFundings response:
#   {"fundingIntervalHours": 8, "nextFundingTime": 1773561600000}
#   Types: int, int

# Bybit tickers response (same concept, different names and types):
#   {"fundingIntervalHour": "8", "nextFundingTime": "1773561600000"}
#   Types: string, string

# Parse defensively -- explicit type conversion for each source
interval = int(rate_data["fundingIntervalHours"])       # Hyperliquid: int key, int value
interval = int(result["fundingIntervalHour"])            # Bybit: singular key, string value
next_time = int(result["nextFundingTime"])               # Bybit: string, needs int()

# Some venues return None for fields that exist on other venues
# Always handle missing/None data per-field, not per-API-call
for venue_name, rate_data in venues:
    if rate_data is None:
        logger.warning("Null data for %s/%s", coin, venue_name)
        continue
    try:
        rate = Decimal(rate_data["fundingRate"])
    except (KeyError, TypeError, InvalidOperation) as e:
        logger.warning("Bad field for %s/%s: %s", coin, venue_name, e)
        continue
```

## 11. Broad Sweep Then Targeted Validation

```python
# Stage 1: one API call returns data for all sources
snapshots = await client.fetch_aggregated_data()  # covers 3 exchanges, 200+ coins

# Stage 2: filter to candidates worth validating
candidates = [s for s in snapshots if s.spread > threshold]  # maybe 3-5 out of 200

# Stage 3: validate only the candidates against direct APIs
# Use a semaphore to limit concurrent validation calls
async with asyncio.Semaphore(5):
    for candidate in candidates:
        validated = await client.validate_with_source_api(candidate)
        if divergence(validated, candidate) > 0.20:
            candidate = recalculate(validated)

# Result: 1 broad API call + 3-5 targeted calls instead of 200+ calls
```
