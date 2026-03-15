---
name: async-safety-checklist
description: Async safety and race condition prevention for Python asyncio. Use when writing code with shared state, background tasks, shutdown handlers, or lock files. Covers asyncio.Lock patterns, atomic file creation, sequential queues, and shutdown ordering. Use when user says "race condition", "data corruption", "two tasks writing same file", or "graceful shutdown".
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

Any data touched by more than one task needs an asyncio.Lock(). This includes file I/O (the classic read-modify-write race). Use per-resource locks when resources are independent -- a single global lock makes independent operations wait unnecessarily.

### 2. Never Cancel a Task From Inside Itself

If Task A detects an event and calls a handler that cancels Task A, the code after the handler call never executes. Decouple detection from handling: the detector sets a flag or enqueues an item, a separate consumer processes it.

### 3. Atomic Lock Files

Prevent duplicate instances with atomic file creation using O_CREAT | O_EXCL. Never use exists() + create() -- that is a TOCTOU race where two processes can pass the check simultaneously.

### 4. Sequential Processing for Order-Dependent Operations

If two operations can race and produce an incorrect combined result, process them through an asyncio.Queue one at a time. This prevents scenarios like two signals both reading stale state and exceeding limits.

### 5. Strict Shutdown Ordering

Stop things in dependency order: producers first (monitors/workers), then schedulers, then communication channels last. If you cancel in arbitrary order, a monitor may try to send through an already-closed channel.

### 6. Never Globally Mock Timing Primitives

In tests, do not replace asyncio.sleep globally. Background loops will spin at infinite speed and crash. Patch sleep only in the specific function being tested.

## Common Mistakes

- Forgetting that file I/O is shared mutable state. Two tasks saving to the same JSON file will corrupt it without a lock.
- Using exists() + create() instead of atomic O_CREAT | O_EXCL for lock files.
- Cancelling tasks in arbitrary order during shutdown, causing ConnectionError or AttributeError.
- Using a single global lock when per-resource locks would allow independent operations to run concurrently.

For code examples, see references/code-examples.md in this skill's directory.
