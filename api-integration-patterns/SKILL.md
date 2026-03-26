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

1. **Price/Data Caching with Short TTL** — Cache responses for 2s+ to avoid hammering.
2. **Skip Redundant Calls** — If local state matches desired state, don't call. Cache the last-set value.
3. **Fuzzy Identifier Matching** — Different systems use different names. Build a mapper with known overrides.
4. **"Not Found" on Cancel = Already Filled** — Treat this as success, not an error.
5. **Testnet First** — Always develop against sandbox. Archive testnet data before migrating.
6. **Retry with Context** — Log what failed and why. Use exponential backoff. Limit retries.
7. **Use SDK Exceptions, Not Raw HTTP** — Catch SDK-provided exceptions (ClientError, ServerError) instead of raw HTTPError.
8. **Track API Weight, Not Call Count** — Many APIs rate-limit on weight. A candle call (weight 20) costs 10x a price check (weight 2). Track and display weight/min.
9. **Audit ALL Polling Loops** — List every background loop that makes API calls, multiply out the weight, sum them. Hidden consumers (fill checkers, health monitors) can double usage.
10. **Same Data, Different Field Names** — Different APIs return the same concept under different names and types. Parse defensively with explicit type conversion. Always verify against the actual response, not the docs.
11. **Broad Sweep Then Targeted Validation** — When one API aggregates data from multiple sources, use it for the initial scan, then hit individual source APIs only for candidates that pass filtering.
12. **Isolate Third-Party SDKs in One File** — All imports of a third-party SDK should live in a single wrapper file. Use lazy imports inside methods (not at module top) so the SDK stays an optional dependency. If the SDK is sync and your codebase is async, wrap calls with `run_in_executor`. When the SDK ships a breaking release, only one file changes.
13. **No-Retry on Irreversible Actions** — For actions that could duplicate (market orders, payments, messages), do NOT retry on failure. Instead: wait briefly, then check if the action took effect. If it did, treat as success. If not, re-raise the error. Retrying risks double execution.
14. **WebSocket Over Polling When Available** — If the API offers WebSocket subscriptions, use them for monitoring instead of REST polling. REST polling weight scales with the number of items monitored. WebSocket is flat-cost regardless of item count. Keep REST as a periodic safety net (e.g., every 20-30s) to catch anything WebSocket missed.

For code examples and detailed patterns, see references/code-examples.md in this skill's directory.

15. **Verify SDK Constructor Defaults** — Third-party SDK constructors may silently disable features via hidden defaults. Example: `Exchange.__init__` passes `skip_ws=True` to `Info.__init__`, so `ws_manager` is always None. Read the SDK source for any constructor you depend on. After init, assert that the objects you need are actually created (`assert info.ws_manager is not None`).
16. **Per-Item Error Handling in Batch Subscriptions** — When subscribing/registering/connecting to multiple items (coins, channels, endpoints), wrap each item in its own try/except. One unavailable item must not abort the remaining items. Log the failure per-item and continue.
17. **Derive Signing Address from Private Key, Verify at Startup** — In crypto exchange APIs, the "wallet address" (for reading) and the "signing address" (derived from the private key) are separate concepts. An API may accept either for read operations but require the registered API wallet for writes. At startup, derive the address from the private key (`Account.from_key(key).address`) and verify it matches expectations. Mismatch causes read-only symptoms: polling works, all order placement fails with "wallet does not exist."
18. **Startup Jitter for Multiple Polling Instances** — When N background pollers start simultaneously (e.g., N=8 monitors recovering together), per-sleep jitter prevents drift over time but does NOT prevent the initial synchronized burst at T=0. Add a per-instance startup offset sampled once at init: `self._startup_offset = random.uniform(0, poll_interval)`. Sleep this offset before the first poll. CDN-layer rate limits (Cloudfront, Fastly) track burst rate per IP, independently of an API's weight-based limits — they can trigger even when weight usage is low.

## Common Mistakes

- Calling get_price() every second when a 2-second cache would cut calls in half.
- Treating "order not found" on cancel as a crash instead of a normal race condition.
- Hardcoding identifier mappings only for today's assets. New assets fail silently.
- Using production API keys during development.
- Tracking calls instead of weight. 150 calls/min can be 1100 weight/min with heavy endpoints.
- Forgetting background polling loops in rate limit budgets. Hidden pollers push you over at scale.
- Importing a third-party SDK in 10+ files. When the SDK ships a breaking API change, you rewrite everywhere instead of one wrapper.
- Trusting SDK constructor defaults without reading the source. A constructor may silently skip WebSocket, caching, or auth init.
- Wrapping batch subscribe/register loops in a single try/except. One bad item kills the rest.
- Assuming read access implies write access. In two-address API auth systems, valid read credentials can coexist with invalid signing credentials for months before a write is attempted.
