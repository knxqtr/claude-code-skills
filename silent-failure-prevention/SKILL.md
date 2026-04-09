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

## Data Pipeline Consistency

When filtering a subset of data for evaluation, ALL inputs to the evaluation must be filtered -- not just the primary data. If you filter trades to out-of-sample-only but pass the full equity curve to a metrics function, the metrics are silently wrong. This applies to any pipeline where:
- One input is filtered (e.g., trades) but a correlated input (e.g., equity curve) is not
- Serialized data omits fields that downstream consumers reference via `dict.get()` with defaults
- A subset is selected for scoring but the scoring function receives unfiltered auxiliary data

The failure mode is always silent: no crash, no error, just plausible-looking wrong numbers.

Watch for `dict.get("key", default)` where the key is expected to exist but doesn't. The default value silently masks the absence. Common case: config dicts where a key was never added but the code assumes it exists. If a threshold depends on a runtime-computed value (like number of folds based on dataset size), call the computation function directly instead of looking up a config key with a fallback.

## State Machine Exception Safety

Generic `except Exception` blocks in state machine drivers must not leave the object in an intermediate state. If a monitor or handler raises unexpectedly, the state machine entry stays wherever it was when the exception fired -- typically a non-terminal state like "processing" or "entering" that no other code path handles. Always transition to a terminal or safe state in the except block (close, cancel, error). Without this, the entry is orphaned: no active handler, no safety net coverage, no user notification.

## State Transitions Must Not Wipe Fields That Other Lookups Depend On

When a state-machine transition replaces a structured field wholesale (e.g. `trade.orders = new_orders`), every downstream lookup that reads a key of that field becomes dead code if the key is dropped by the replacement. This is silent: the lookup compiles, runs, and always returns None, so any handler gated on it is never invoked. The feature appears to ship because tests manually set the field post-hoc and never exercise the post-transition state.

Before any wholesale field replacement in a transition, list every downstream site that reads keys from the field. If any of them are expected to work after the transition, merge those keys back in, or keep the field append-only and only add new keys. Add at least one regression test that exercises the downstream lookup *after* driving the real transition (not by manually setting the field on a fresh fixture).

Red flag: a feature ships with passing tests, then never fires in production. Check whether the test fixtures reproduce a state the real code path could actually reach, or whether they skip the transition entirely.

## Auto-Remediation Must Notify on Success, Not Only Failure

Reconciliation paths that silently correct state divergence (residual position cleanup, orphan order cancellation, tracker/exchange size reconciliation) must emit a user-visible notification on successful remediation, not only on failure. A silent successful remediation swallows the very thing it was remediating -- if the residual came from real unreported activity, the operator never sees that the activity existed. The classic failure mode: a bug upstream causes tracker/exchange drift, the reconciler closes the residual without notification, and the PnL from that residual vanishes from the audit trail.

Minimum bar: emit an alert ("auto-closed residual X on Y after Z; reconcile manually") whenever the reconciler takes corrective action. Full per-event accounting can follow, but the visibility layer comes first.

Distinguish from the case where the reconciler merely confirms expected state (no action taken). Silence there is fine. The rule is: any corrective action requires a notification.

## Sanitize Error Messages Before Formatted Notifications

When sending error messages through a formatted channel (HTML, Markdown), the error content itself may contain markup that breaks the formatter. Exchange APIs returning HTML error pages (502, 503), stack traces containing angle brackets, or user input with special characters will cause the notification to be rejected by the downstream service (e.g., Telegram rejects unparseable HTML).

Always escape or sanitize error message content before embedding it in a formatted template. In Python: `html.escape(error_msg)` before wrapping in `<code>` tags. If the formatted send fails, fall back to plain text mode rather than retrying the same broken message.

This matters most during outages, when error notifications are most critical. A notification system that fails when the thing it monitors fails is worse than no notification system.

## Common Mistakes

- Catching exceptions in a framework error handler that only logs. The user sees the "processing" message but never gets a result.
- Only wrapping one call site when the same function is called from multiple places.
- Retrying the same failing call as the only "fallback" strategy.
- No wait period before alternative method. Sometimes the original failure self-resolves.
- Serializing data without all fields the consumer needs. `dict.get("missing_key", "")` silently returns the default, producing wrong results instead of a crash.
- Filtering primary data but not auxiliary data in a multi-input evaluation (e.g., filtering trades but not the equity curve).
- Generic except blocks that log but leave state machines in intermediate states. The entry stays orphaned forever.

For code examples, see references/code-examples.md in this skill's directory.
