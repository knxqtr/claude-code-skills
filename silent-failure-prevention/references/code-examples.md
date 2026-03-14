# Silent Failure Prevention — Code Examples

## The Feedback Wrapper

```python
try:
    result = await do_the_thing(params)
    await notify_user_success(result)
except Exception as e:
    logger.error(f"Operation failed: {e}")
    await notify_user_failure(str(e))  # THIS is the critical line
```

Apply this at every call site, not just one.

## Bad Fallback vs Good Fallback

Bad fallback (just retries the same thing):
```
Attempt 1: call API endpoint X  -- fails
Attempt 2: call API endpoint X  -- fails
Attempt 3: call API endpoint X  -- fails
Give up.
```

Good fallback (uses a different code path):
```
Attempt 1-3: call API endpoint X  -- fails
Notify user: "Primary method failed, trying alternative"
Wait period: give time for external recovery or manual intervention
Check: did the problem resolve itself while we waited?
Alternative: call API endpoint Y (different code path)
If still fails: urgent alert with manual instructions
```

## Dead Man's Switch — Full Steps

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
