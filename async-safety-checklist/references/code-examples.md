# Async Safety Checklist — Code Examples

## 1. Shared Mutable State Lock Pattern

```python
# The classic read-modify-write race:
# Task A reads file -> Task B reads file (stale) ->
# Task A writes -> Task B writes (clobbers A's changes)

_lock = asyncio.Lock()

async def save_state(key, value):
    async with _lock:
        data = read_json(path)     # read
        data[key] = value          # modify
        write_json(path, data)     # write
```

## 2. Decouple Detection From Handling

```python
# BAD: monitor detects event -> calls handler -> handler cancels monitor
#      -> notification code after handler call never runs

# GOOD: monitor sets a flag, separate handler loop processes it
```

## 3. Atomic Lock Files

```python
# BAD: TOCTOU race (two processes can pass the check simultaneously)
if not os.path.exists(lock_file):
    open(lock_file, 'w').write(pid)

# GOOD: atomic create-or-fail
fd = os.open(lock_file, os.O_CREAT | os.O_EXCL | os.O_WRONLY)
os.write(fd, str(os.getpid()).encode())
os.close(fd)
```

## 4. Sequential Processing via Queue

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

## 5. Shutdown Ordering

```
1. Stop monitors/workers      -- may send final notifications
2. Stop schedulers            -- heartbeat, summaries
3. Stop communication channel -- disconnect last
```

## 6. Targeted Sleep Mocking

```python
# BAD: global autouse fixture that replaces asyncio.sleep with instant no-op
# Result: background loops run millions of iterations per second

# GOOD: patch sleep only in the specific function being tested
with patch.object(my_module, 'asyncio.sleep', return_value=None):
    await test_specific_function()
```
