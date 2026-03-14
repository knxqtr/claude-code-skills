---
name: crash-recovery-design
description: Crash recovery and state persistence for long-running services. Use when building bots, daemons, or workers that must survive restarts without losing active work. Covers startup/shutdown ordering, state file design, and detecting events missed while offline. Use when user says "loses state", "forgets after restart", or "what if it crashes".
---

# Crash Recovery Design

## When to Use

- Building any long-running service (bot, daemon, worker, scheduler)
- Service manages active work items that must survive restarts
- Service talks to an external system (API, exchange, database) that has its own state
- You need to answer: "What happens if this process dies right now?"

## The Three-Case Recovery Pattern

On startup, compare your local state file against the external system. Every item falls into one of three cases:

### Case A: In local state AND in external system
Full restore. Rebuild the in-memory object from saved state. Resume all monitoring (timers, health checks, watchers). Check for events that happened while offline. Notify the user that monitoring has resumed.

### Case B: In local state but NOT in external system
The item completed or was removed while you were offline. Notify the user what happened. Clean up local state (remove from file).

### Case C: In external system but NOT in local state
Unknown item with no history. Create a minimal tracking record with safe defaults. Populate ALL fields the rest of the code expects (not just the obvious ones). Alert the user so they can provide missing context.

## State File Rules

1. Write immediately on every state change. Do not batch or delay writes.
2. Include everything needed to fully reconstruct the in-memory object.
3. Use a human-readable format (JSON) so you can debug by reading the file.
4. Protect writes with a lock if multiple async tasks can modify state.

For startup ordering, shutdown ordering, and offline event detection details, see `references/startup-shutdown.md`.

## Common Mistakes

- Case C missing fields: code assumes fields exist that only Case A populates. Every code path that touches an item must work with Case C's minimal record. Populate ALL fields with defaults.
- Recovery before notification channel: notifications silently fail because the channel is not connected yet.
- No lock file: two instances run simultaneously, duplicating all actions. Use atomic lock files (O_CREAT | O_EXCL, not exists() + create()).
- Batched writes: process dies between state change and next write cycle, losing the change. Write immediately.
