# Startup and Shutdown Ordering

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
