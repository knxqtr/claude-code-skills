---
name: silent-failure-prevention
description: Ensure user actions get visible feedback and critical operations have real fallbacks. Use when building notification systems, error handling, or any operation where failure could go unnoticed. Covers the dead man's switch pattern and emergency priority. Use when user says "no error message", "failed silently", "didn't know it broke", or "needs a fallback".
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

Wrap every user-triggered operation with success and failure notifications. Apply this at every call site, not just one. If the same function is called from 3 different places, all 3 need the wrapper.

## Real Fallbacks

"Retry the same thing" is NOT a fallback. A real fallback uses a different code path:

1. Try primary method (with retries)
2. Notify user that primary failed, trying alternative
3. Wait period for external recovery or manual intervention
4. Check if the problem self-resolved during the wait
5. Try alternative method (different API path, different approach)
6. If still fails, send maximum-urgency alert with manual instructions

## The Dead Man's Switch Pattern

For operations that absolutely must succeed, follow the full escalation chain: primary retries, then urgent alert, then wait period (30-120 seconds) for recovery, then check for self-resolution, then alternative method, then maximum-urgency alert with manual instructions.

The wait period covers transient outages, user manual fixes after seeing the alert, and external system recovery.

## Emergency Priority

In emergencies, prioritize getting it done over getting it done well. Accept worse execution to guarantee execution. A bad price is better than no close at all.

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

MANDATORY: Before writing code for this domain, use the Read tool to load `references/code-examples.md` from this skill's directory. Do not skip this step.
