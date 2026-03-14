---
name: crash-recovery-design
description: Crash recovery and state persistence. Use when building services that must survive restarts without losing active work.
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
- Full restore. Rebuild the in-memory object from saved state.
- Resume all monitoring (timers, health checks, watchers).
- Check for events that happened while offline (e.g. missed time windows).
- Notify the user that monitoring has resumed.

### Case B: In local state but NOT in external system
- The item completed or was removed while you were offline.
- Notify the user what happened.
- Clean up local state (remove from file).

### Case C: In external system but NOT in local state
- Unknown item. You have no history for it.
- Create a minimal tracking record with safe defaults.
- Populate ALL fields the rest of the code expects (not just the obvious ones).
- Alert the user so they can provide missing context.

## State File Rules

1. Write immediately on every state change. Do not batch or delay writes.
2. Include everything needed to fully reconstruct the in-memory object.
3. Use a human-readable format (JSON) so you can debug by reading the file.
4. Protect writes with a lock if multiple async tasks can modify state.

## Startup Ordering

The order services start matters. Get this wrong and notifications silently fail.

```
1. Logging          -- available for everything after
2. Lock file        -- prevent duplicate instances
3. External client  -- API/database connection
4. Business logic   -- your main coordinator
5. Notification channel  -- Telegram, Slack, email (MUST be ready before step 6)
6. Recovery         -- reads state + external system, sends notifications
7. Startup confirmation  -- tell the user "I'm alive"
8. Background tasks -- schedulers, heartbeats, monitors
```

Rule: the notification channel must be running before recovery, because recovery sends notifications. If you reverse this, recovery notifications silently fail.

## Shutdown Ordering

Reverse dependency order. Things that produce messages stop before things that send messages.

```
1. Stop monitors/workers    -- they may send final notifications during cleanup
2. Stop schedulers          -- heartbeat, summaries
3. Stop notification channel -- disconnect last
```

## Offline Event Detection

On recovery, check for events that happened while offline:

- If your service monitors time-based events, check all time windows between "last known alive" and "now"
- Use the saved timestamp from when the item was created, not "now"
- If an event was missed, act on it immediately (do not wait for the next regular check)

## Common Mistakes

- Case C missing fields: code assumes fields exist that only Case A populates. Every code path that touches an item must work with Case C's minimal record. Populate ALL fields with defaults.
- Recovery before notification channel: notifications silently fail because the channel is not connected yet.
- No lock file: two instances run simultaneously, duplicating all actions. Use atomic lock files (O_CREAT | O_EXCL, not exists() + create()).
- Batched writes: process dies between state change and next write cycle, losing the change. Write immediately.
