---
name: async-safety-checklist
description: Race condition prevention and async safety patterns for Python asyncio projects. Use when writing concurrent code, background tasks, shared state, file I/O, or shutdown handlers. Use when user says "async", "race condition", "concurrent", "lock", "asyncio", "background task", or "shutdown order".
---

# Async Safety Checklist

## When to Use

- Writing any asyncio Python code with shared state
- Multiple tasks reading/writing the same file or data structure
- Background loops (polling, monitoring, health checks)
- Implementing graceful shutdown
- Preventing duplicate process instances

## Checklist

Run through this list for every async project:

### 1. Shared Mutable State Needs a Lock

Any data touched by more than one task needs an asyncio.Lock(). This includes file I/O.

```python
# The classic read-modify-write race:
# Task A reads file → Task B reads file (stale) →
# Task A writes → Task B writes (clobbers A's changes)

_lock = asyncio.Lock()

async def save_state(key, value):
    async with _lock:
        data = read_json(path)     # read
        data[key] = value          # modify
        write_json(path, data)     # write
```

Use per-resource locks when resources are independent. A single global lock makes independent operations wait for each other unnecessarily.

### 2. Never Cancel a Task From Inside Itself

If Task A detects an event and calls a handler, and that handler cancels Task A, the code after the handler call never executes.

```python
# BAD: monitor detects event → calls handler → handler cancels monitor
#      → notification code after handler call never runs

# GOOD: monitor sets a flag, separate handler loop processes it
```

Decouple detection from handling. The detector sets a flag or puts an item on a queue. A separate consumer processes it.

### 3. Atomic Lock Files

Prevent duplicate instances with atomic file creation:

```python
# BAD: TOCTOU race (two processes can pass the check simultaneously)
if not os.path.exists(lock_file):
    open(lock_file, 'w').write(pid)

# GOOD: atomic create-or-fail
fd = os.open(lock_file, os.O_CREAT | os.O_EXCL | os.O_WRONLY)
os.write(fd, str(os.getpid()).encode())
os.close(fd)
```

### 4. Sequential Processing for Order-Dependent Operations

If two operations can race and produce an incorrect combined result, process them sequentially.

```python
# BAD: two signals arrive simultaneously, both read leverage as 3x,
#      both place trades, combined leverage exceeds 4x limit

# GOOD: process signals through a queue, one at a time
_signal_queue = asyncio.Queue()

async def signal_processor():
    while True:
        signal = await _signal_queue.get()
        await process_signal(signal)  # checks leverage with current state
```

### 5. Strict Shutdown Ordering

Things that produce messages stop before things that send messages.

```
1. Stop monitors/workers      -- may send final notifications
2. Stop schedulers            -- heartbeat, summaries
3. Stop communication channel -- disconnect last
```

If you cancel in arbitrary order, a monitor may try to send a notification through an already-closed channel.

### 6. Never Globally Mock Timing Primitives

In tests, do not replace asyncio.sleep globally. Background loops that call sleep in a tight loop will spin at infinite speed, consuming 100% CPU and crashing.

```python
# BAD: global autouse fixture that replaces asyncio.sleep with instant no-op
# Result: background loops run millions of iterations per second

# GOOD: patch sleep only in the specific function being tested
with patch.object(my_module, 'asyncio.sleep', return_value=None):
    await test_specific_function()
```

## Common Mistakes

- Forgetting that file I/O is shared mutable state. Two tasks saving to the same JSON file will corrupt it without a lock.
- Using exists() + create() instead of atomic O_CREAT | O_EXCL for lock files.
- Cancelling tasks in arbitrary order during shutdown, causing ConnectionError or AttributeError.
- Using a single global lock when per-resource locks would allow independent operations to run concurrently.
