# Skill Extraction Log

Records what was considered, kept, and skipped during each skill extraction. Review periodically to spot patterns -- if the same lesson gets skipped 3+ times, the existing skill may need fixing.

---

## FATB, v3.4.3 (2026-04-09)
- UPDATED: silent-failure-prevention — added "State Transitions Must Not Wipe Fields That Other Lookups Depend On" (RC-42 Pattern 1: confirm_entry wiped trade.orders["entry"], silently disabling the v1.11.22 late-entry-fill handler for months) and "Auto-Remediation Must Notify on Success, Not Only Failure" (RC-42 Pattern 3: _reconcile_position closed 3.55 HYPE residual silently)
- SKIPPED: delta-based idempotent fill dispatch (RC-42 Pattern 2) — specific implementation pattern, already implicitly covered by "use authoritative state, not per-event payload"

## FATB, v3.2.2 (2026-04-06)
- UPDATED: async-safety-checklist — added rule 8 "Guard Set Entry Must Precede the First Await in the Guarded Block" (RC-39)
- SKIPPED: duplicate metrics display (RC-40) — project-specific monitoring display behavior, not transferable

## FATB, v3.2.0 (2026-04-05)
- UPDATED: order-management-edge-cases — added "WebSocket Reconnect Fill Dedup" section (RC-36) and "Exit Dispatch Must Not Depend Solely on Order Status" section (RC-37)
- UPDATED: silent-failure-prevention — added "Sanitize Error Messages Before Formatted Notifications" section (RC-38)
- SKIPPED: startup leverage skip (RC-37 fix #4) — project-specific Hyperliquid API behavior, not transferable
- SKIPPED: WS resubscribe fast-fail (RC-37 fix #5) — SDK-specific reconnection quirk, not transferable

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

## Trading Bot, v3.5.0 (2026-03-25)
- UPDATED crash-recovery-design: added "Redundant State Backup" section — back up state file to an already-integrated external service to survive full server loss
- SKIPPED: v3.4.0 /closelongs /closeshorts — project-specific feature, no transferable pattern
- SKIPPED: v3.4.1 TP2 PnL breakdown — project-specific notification formatting

## FATB, v3.1.1 (2026-04-04)
- UPDATED mock-fidelity-standards Rule 5: clarified that shared fixtures belong in conftest.py (not test files), and that file-scoped fixtures produce ERROR at setup rather than FAILED, causing broken tests to silently appear to pass
- SKIPPED: WS health poll 30s → 1s -- implementation detail of thread-based WebSocket SDK health checks, not generalizable
- SKIPPED: git local core.hooksPath override -- too narrow; single-sentence fix, no existing skill to add to

## Trading Bot, v3.5.1 (2026-03-26)
- UPDATED api-integration-patterns: added pattern #17 "Derive Signing Address from Private Key, Verify at Startup" (AX-1: API wallet address mismatch — reads fine, writes fail with "wallet does not exist")
- UPDATED api-integration-patterns: added pattern #18 "Startup Jitter for Multiple Polling Instances" (TP-4: synchronized startup of 8 monitors caused Cloudfront 429 bursts every ~12 minutes; per-sleep jitter alone doesn't prevent initial phase alignment)
- UPDATED crash-recovery-design: added common mistake about synthetic marker values as recovery discriminators (RP-4: strategy="recovered" blocked Case A on subsequent restart)

## signal-backtester, v4.3.0 (2026-04-04)
- NEW SKILL: backtesting-validation — WFT window isolation (simulate per OOS window, not global simulate + filter), holdout must use same engine as training, save all pipeline outputs to files before downstream runs. No existing skill covered WFT simulation correctness.
- UPDATED financial-math-rules: added common mistake about taker vs maker fee drag. At 100+ trades/month, the difference can exceed a strategy's entire edge.
- SKIPPED: JIT/numba cold-start benchmark lesson (too niche, only applies to projects using native JIT compilation)
- SKIPPED: Exit A vs Exit D holdout durability (trading strategy evaluation, too domain-specific)
- SKIPPED: Regime sensitivity of WFT OOS results (trading-specific, no generalizable code pattern)
- SKIPPED: Walk-forward persistent executor bug (v3.4.29) — project-specific wiring issue, not a generalizable pattern
## signal-backtester, v4.6.25 (2026-04-07)
- SKIPPED: "robust_map must report unsubmitted tasks and completed failure futures on timeout/broken submit paths" -- already covered by existing skills `silent-failure-prevention` (no silent drops in multi-input pipelines) and `crash-recovery-design` (full accounting for in-flight work under failure).

## signal-backtester, v4.6.27 (2026-04-07)
- SKIPPED: "keep three robust ordered-batch helpers duplicated until a fourth copy appears or semantics converge" -- project-specific maintenance threshold, not a reusable cross-project skill.

## FATB, v3.3.0 (2026-04-08)
- SKIPPED: limit entry retrace feature -- new feature implementation, not a bug fix or non-obvious pattern; the architecture (metadata bag for derived order prices, keeping original entry for cancel checks) is already covered by order-management-edge-cases and financial-math-rules
- SKIPPED: defense-in-depth validation at both config and helper -- general good practice, fails quality gate test 3

## FATB, v3.2.3 (2026-04-07)
- SKIPPED: "reset polling throttle timer after fallback REST check confirms target alive" -- single occurrence, covered by existing defense-in-depth-monitoring rules (confirmation before acting on disappearances, alert deduplication).

## signal-backtester, v4.6.43 (2026-04-09)
- SKIPPED: "validate empirical presets before applying runtime sidecar overrides so stale documented stop_pct values still warn" -- already covered by existing `silent-failure-prevention` pipeline-consistency guidance; project fix was implementation of the guardrail, not a new cross-project skill category.

## Trading Bot, v3.6.0 (2026-04-13)
- SKIPPED: "async replay backtester must await shared retry helper" -- project-specific offline tooling fix; no reusable rule beyond existing async-safety and backtesting-validation skills.
- SKIPPED: "label offline replay as approximate instead of live-equivalent" -- product/tooling scope decision, captured in PROJECT_HISTORY.md but not broad enough for a new or updated generic skill.

## Trading Bot, v3.6.1 (2026-04-13)
- SKIPPED: "deploy-generated sudoers file must be chowned to root after install" -- specific to this deploy script and sudo drop-in workflow; captured in PROJECT_HISTORY.md, not broad enough for a new or updated generic skill.
