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
