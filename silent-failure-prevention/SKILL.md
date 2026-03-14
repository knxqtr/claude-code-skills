---
name: silent-failure-prevention
description: Ensure user actions get visible feedback and critical operations have real fallbacks. Use when failures could go unnoticed.
---

# Silent Failure Prevention

## When to Use

- Any user-triggered operation that could fail
- Any critical operation that must succeed (closing positions, sending payments, stopping services)
- Any background process where the user expects a result
- Building notification or alerting systems

## Core Rule

Logging to a file is NOT the same as telling the user. The user is not watching the log file at 3am. Every user-triggered operation must notify through the user-facing channel on failure.

## The Feedback Pattern

Wrap every user-triggered operation:

```python
try:
    result = await do_the_thing(params)
    await notify_user_success(result)
except Exception as e:
    logger.error(f"Operation failed: {e}")
    await notify_user_failure(str(e))  # THIS is the critical line
```

Apply this at every call site, not just one. If the same function is called from 3 different places, all 3 need the wrapper.

## Real Fallbacks

"Retry the same thing" is NOT a fallback. A real fallback uses a different code path.

Bad fallback:
```
Attempt 1: call API endpoint X  -- fails
Attempt 2: call API endpoint X  -- fails
Attempt 3: call API endpoint X  -- fails
Give up.
```

Good fallback:
```
Attempt 1-3: call API endpoint X  -- fails
Notify user: "Primary method failed, trying alternative"
Wait period: give time for external recovery or manual intervention
Check: did the problem resolve itself while we waited?
Alternative: call API endpoint Y (different code path)
If still fails: urgent alert with manual instructions
```

## The Dead Man's Switch Pattern

For operations that absolutely must succeed:

1. Try the primary method (with retries)
2. If all retries fail, send urgent alerts to the user
3. Wait a defined period (30-120 seconds) for external recovery or manual intervention
4. Check if the problem resolved itself during the wait
5. If still unresolved, try an alternative method (different API path, different approach)
6. If alternative also fails, send maximum-urgency alert with manual instructions

The wait period is not wasted time. It covers:
- Transient outages that self-resolve
- The user manually fixing it after seeing the alert
- External systems recovering from temporary issues

## Emergency Priority

In emergencies, prioritize getting it done over getting it done well. Accept worse execution to guarantee execution.

Example: if a normal close fails, use a wider-slippage emergency close. A bad price is better than no close at all.

## Checklist

For every user-facing operation, verify:
- [ ] Success path sends confirmation to user
- [ ] Failure path sends error to user (not just logs)
- [ ] All call sites are wrapped (not just one)
- [ ] Critical operations have a fallback using a different code path
- [ ] Fallback failure triggers maximum-urgency alert
- [ ] Emergency operations prioritize completion over quality

## Common Mistakes

- Catching exceptions in a framework error handler that only logs. The user sees the "processing" message but never gets a result.
- Only wrapping one call site when the same function is called from multiple places.
- Retrying the same failing call as the only "fallback" strategy.
- No wait period before alternative method. Sometimes the original failure self-resolves.
