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

## Common Mistakes

- Calling get_price() every second when a 2-second cache would cut calls in half.
- Treating "order not found" on cancel as a crash instead of a normal race condition.
- Hardcoding identifier mappings only for today's assets. New assets fail silently.
- Using production API keys during development.
- Tracking calls instead of weight. 150 calls/min can be 1100 weight/min with heavy endpoints.
- Forgetting background polling loops in rate limit budgets. Hidden pollers push you over at scale.
- Importing a third-party SDK in 10+ files. When the SDK ships a breaking API change, you rewrite everywhere instead of one wrapper.
