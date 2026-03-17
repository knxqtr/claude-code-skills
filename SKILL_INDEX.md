# Skill Index

Keyword-to-skill mapping for skill-loader. Before writing implementation code, scan this list for keywords that match what you're about to write. If any match, load that skill.

## Domain Skills

retry, backoff, 429, rate limit, polling loop, API weight, caching, TTL, external API, SDK exception, identifier mapping, SDK wrapper, lazy import, run_in_executor, optional dependency -> api-integration-patterns

asyncio.Lock, background task, shared state, race condition, graceful shutdown, lock file, task cancellation, shutdown ordering, sequential queue -> async-safety-checklist

restart, crash recovery, state file, loses state, daemon, long-running service, startup ordering, shutdown ordering -> crash-recovery-design

health check, alert, missed event, duplicate alert, false trigger, price trigger, multi-layer detection, dead man's switch, time guard -> defense-in-depth-monitoring

Decimal, money, PnL, fee, balance, rounding, fill price, slippage, taker rate, maker rate, half-open interval -> financial-math-rules

mock, fake, stub, test simulator, error fidelity, partial fill, cross-cutting failure, timing mock -> mock-fidelity-standards

fuzz test, Hypothesis, random testing, state machine testing, invariant, event generation, simulated clock -> property-based-testing

silent failure, no error message, failed silently, fallback, notification, user feedback, emergency priority -> silent-failure-prevention

VPS, systemd, rsync, deployment, remote server, auto-restart, service unit, heartbeat -> vps-deployment-checklist

partial fill, cancel race, order resize, order lifecycle, limit order, market order -> order-management-edge-cases

research, deep dive, unfamiliar domain, new feature design, domain knowledge, parallel agents, research agents, spec preparation, design research -> research-skill

## Maintenance

When creating or updating a skill during extraction, update this index with relevant keywords.
