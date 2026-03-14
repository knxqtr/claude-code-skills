---
name: defense-in-depth-monitoring
description: Multi-layer event detection for reliable monitoring. Use when building health checks, alerts, or any system where missing an event has consequences.
---

# Defense in Depth Monitoring

## When to Use

- Monitoring for time-sensitive events (price triggers, SLA breaches, health checks)
- Any system where missing an event has real consequences
- Building alerting or notification systems
- Detecting state changes in external systems

## The Three-Layer Detection Pattern

No single detection method is 100% reliable. Use three layers:

### Layer 1: Precise Timer (lowest latency)
Calculate the exact time the event should occur. Sleep until that time, then check. Add a small buffer (2-3 seconds) for delays. Fragile: depends on knowing the exact schedule.

### Layer 2: Regular Polling (medium latency)
Check at fixed intervals (15-60 seconds) regardless of the timer. Catches events the timer missed. Works for non-standard schedules.

### Layer 3: Safety Net (highest latency)
If you expected data but did not get it, fall back to checking raw state. Require multiple consecutive confirmations (2+ checks 15 seconds apart) to guard against stale or glitchy data.

When Layer 1 fires, cancel the next Layer 2 check to avoid duplicate processing.

## Key Rules

- Time guards on historical data: after a restart, only consider events that happened AFTER the relevant start time. Filtering prevents false triggers on pre-existing data.
- Non-standard interval mapping: if your data source does not support the exact interval, map to the nearest smaller supported interval and check more frequently (e.g., need 45 min, use 30 min and check 2x).
- Alert deduplication: health checks on a loop must track whether they already alerted for the current issue. Alert once, set a flag, reset when resolved.
- Confirmation before acting on disappearances: when something vanishes from an external system, wait and re-check before acting. API glitches can return empty results temporarily.

For detailed code examples of time guards, alert dedup, and confirmation patterns, see `references/code-examples.md`.

## Common Mistakes

- Single-layer detection: one polling loop that can miss events if timing is unlucky.
- Checking historical data without filtering by start time. Causes false triggers on data that predates the monitored item.
- Alerting every poll cycle instead of once per issue. User gets 60 alerts per hour.
- Acting immediately on a disappearance without confirmation. API glitches cause false positives.
