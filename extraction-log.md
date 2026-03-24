# Skill Extraction Log

Records what was considered, kept, and skipped during each skill extraction. Review periodically to spot patterns -- if the same lesson gets skipped 3+ times, the existing skill may need fixing.

---

## trading-bot, v3.2.2 (2026-03-14)
- NEW SKILL: crash-recovery-design — "no existing skill covered state persistence and startup recovery"
- NEW SKILL: silent-failure-prevention — "no existing skill covered user feedback and fallback patterns"
- NEW SKILL: async-safety-checklist — "no existing skill covered race conditions and async safety"
- NEW SKILL: financial-math-rules — "no existing skill covered decimal math and fee handling"
- NEW SKILL: mock-fidelity-standards — "no existing skill covered test mock quality"
- NEW SKILL: defense-in-depth-monitoring — "no existing skill covered multi-layer event detection"
- NEW SKILL: api-integration-patterns — "no existing skill covered API retry and caching"
- NEW SKILL: project-bootstrapping — "no existing skill covered project setup from scratch"
- NEW SKILL: vps-deployment-checklist — "no existing skill covered VPS deployment"
- NEW SKILL: order-management-edge-cases — "no existing skill covered order lifecycle edge cases"
- NEW SKILL: property-based-testing — "no existing skill covered fuzz testing with invariants"
- NEW SKILL: project-history-documentation — "no existing skill covered technical build reference creation"
- NEW SKILL: skill-extraction — "no existing skill covered extracting skills from project history"
- Note: initial extraction, all skills were new. Future extractions should mostly update existing skills.

## trading-bot, v3.3.2 (2026-03-15)
- UPDATED api-integration-patterns — added "Track API Weight, Not Call Count" pattern and "Audit ALL Polling Loops" pattern. Root cause: pending limit fill monitors (2s poll, 2 calls each) were invisible in rate limit calculations, causing 429s overnight with 8 positions despite candle interval fix.
- REVIEWED lessons learned — added 3 exchange integration entries (weight vs calls, hidden polling loops, rename grep discipline)
- SKIPPED: no new skills needed. All learnings fit within existing api-integration-patterns skill.

## funding-arb-bot, v0.1.0 (2026-03-15)
- REVIEWED bug SF-1 (subagent disk fill) — already covered by silent-failure-prevention, no update needed
- REVIEWED decisions (Python 3.13, perp-perp strategy, Bybit, GARCH priority) — project-specific, not generalizable
- SKIPPED: no new code written yet, no new patterns to extract. Re-evaluate at v0.2.0 when scanner is built.

## tao-miner, v0.2.0 (2026-03-16)
- UPDATED api-integration-patterns: added pattern #12 "Isolate Third-Party SDKs in One File" (lazy imports, run_in_executor for sync SDKs, single file for breaking API changes) and corresponding common mistake
- SKIPPED: L5 "do async from the start" -- obvious to experienced devs, fails quality gate test 3
- SKIPPED: L6 "volatile chain data in cache not DB" -- already covered by api-integration-patterns pattern #1 (Price/Data Caching with Short TTL)
- SKIPPED: D4/D5/D6 (async DB, metagraph separation, SDK wrapping decisions) -- project-specific, generic versions captured in L5/L6/L7

## tao-miner, v0.5.0 (2026-03-16)
- UPDATED vps-deployment-checklist: added rule #4 "Secrets must not be CLI arguments" (from B005: Telegram token in ps output) and rule #5 "Uploaded scripts must live inside the package" (from lesson 10: wheel install breaks relative paths)
- SKIPPED: B004 "Literal type coverage" -- grep for assignment sites is good practice but fails quality gate test 3 (most devs already do this)
- SKIPPED: B006/L9 "trace CLI flags end-to-end" -- too generic, fails quality gate test 3 (obvious to experienced devs)
- SKIPPED: L11 "check nullable DB returns" -- partially covered by crash-recovery-design Case C (missing fields)

## strategy-backtester, v1.0.8 (2026-03-19)
- UPDATED financial-math-rules: expanded rule #5 (rounding) with Python round() vs int() cross-platform note. From B021 (HMA banker's rounding vs TradingView truncation).
- SKIPPED: B018/B019 (short condition double-inversion, exit inversion missed) -- "uniform transformation on non-uniform inputs" is a general programming pitfall, not domain-specific enough for a skill. The financial-math-rules rule #13 (mirror short exits) already covers the oscillator-specific case.
- SKIPPED: B020 (stale _prev_row) -- "shared state updated in one lifecycle phase" is a general state management bug. Too generic for a skill rule.
- SKIPPED: B022 (robustness param name mismatch) -- project-specific parameter naming issue. The general lesson (parameter mapping completeness) is too obvious to pass quality gate test 3.
- SKIPPED: B023 (early return blocking recovery) -- general control flow issue. "Don't put recovery code after an early return that blocks it" fails quality gate test 3.

## strategy-backtester, v1.0.2 (2026-03-18)
- UPDATED financial-math-rules: added rules #12 (separate NaN sources in indicators), #13 (mirror short exits correctly), #14 (include all costs in per-trade PnL), plus 4 new common mistakes. From B014 (fillna warmup), B015 (inverted short exits), entry commission fix.
- UPDATED silent-failure-prevention: added dict.get config fallback pattern to Data Pipeline Consistency section. From B017 (num_folds hardcoded to 5 via missing config key).
- SKIPPED: B016 (_current_bar_index misalignment) -- project-specific bar indexing issue, not generalizable beyond backtesters
- SKIPPED: Pine Script syntax fixes -- platform-specific quirks, not generalizable (already noted in financial-math-rules lesson about Pine v5 availability)

## FATB, v0.1.0 (2026-03-24)
- ADDED to api-integration-patterns: "No-Retry on Irreversible Actions -- check position instead of retrying market orders"
- ADDED to api-integration-patterns: "WebSocket Over Polling When Available -- flat-cost monitoring vs linear REST polling"
- SKIPPED: "pandas datetime64[us] vs [ns]" -- too framework-version-specific, narrow applicability
- SKIPPED: "numpy booleans fail identity checks" -- too framework-specific, narrow applicability
- SKIPPED: "Startup ordering: Telegram before recovery" -- already covered by crash-recovery-design common mistakes
- SKIPPED: "Resample ourselves vs use exchange data" -- project-specific decision, not a reusable pattern
