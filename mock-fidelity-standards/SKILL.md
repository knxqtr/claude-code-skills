---
name: mock-fidelity-standards
description: Test mock quality standards. Use when writing mocks, fakes, or simulators to ensure they reproduce real error behavior.
---

# Mock Fidelity Standards

## When to Use

- Writing any mock, fake, or stub for testing
- Building test simulators or harnesses
- Reviewing existing test infrastructure for reliability
- Tests are passing but production is failing (false confidence)

## Core Principle

Mocks must reproduce error behavior, not just success behavior. A lenient mock gives false confidence. If production would raise an exception, the mock must raise the same exception under the same conditions.

## Rules

### 1. Error Behavior Must Match Production

```python
# BAD: mock silently accepts invalid input
class FakeBroker:
    async def place_order(self, size):
        return {"status": "ok", "size": size or 1.0}  # defaults to 1.0!

# GOOD: mock raises like production would
class FakeBroker:
    async def place_order(self, size):
        if size is None or size <= 0:
            raise ValueError("Invalid order size")
        return {"status": "ok", "size": size}
```

### 2. Cross-Cutting Concerns Must Cover ALL Methods

When adding a failure mode (network simulation, rate limiting, auth checks) to a mock, audit EVERY public method. Missing even one creates a hole where tests pass but production fails.

```python
# BAD: added _check_fail() to place_order and get_position,
#      but forgot get_open_orders and get_account_value
#      Result: those methods succeed during simulated outage

# GOOD: add to every public method
class FakeExchange:
    def _check_fail(self):
        if self._should_fail:
            raise ConnectionError("Exchange unreachable")

    async def place_order(self, ...):
        self._check_fail()  # present
        ...

    async def get_position(self, ...):
        self._check_fail()  # present
        ...

    async def get_open_orders(self, ...):
        self._check_fail()  # present -- don't forget this one
        ...

    async def get_account_value(self):
        self._check_fail()  # present -- or this one
        ...
```

### 3. Partial Operations Are Not Full Operations

If the real system can partially complete an operation (partial fill, partial delete, partial write), the mock must model this correctly.

```python
# BAD: TP1 fill removes entire position
def handle_tp_fill(self, order):
    del self.positions[order.coin]  # gone entirely

# GOOD: reduce position by filled amount
def handle_tp_fill(self, order):
    pos = self.positions[order.coin]
    pos.size -= order.filled_size
    if pos.size <= 0:
        del self.positions[order.coin]
```

"Reduce by X" and "remove entirely" are fundamentally different operations.

### 4. Never Globally Mock Timing Primitives

Replacing asyncio.sleep or time.sleep globally makes background loops spin at infinite speed.

```python
# BAD: global fixture replaces sleep for all tests
@pytest.fixture(autouse=True)
def fast_sleep():
    with patch('asyncio.sleep', return_value=None):
        yield
# Result: any background loop runs millions of iterations, 100% CPU, crash

# GOOD: scope patches to specific functions, not global background loops
```

### 5. Deduplicate Test Utilities Early

When 3 or more test files share the same mock class, extract it to a shared helper immediately. Duplicated mocks drift apart over time, creating inconsistent test behavior.

```python
# Before: 23 files with identical FakeMonitor class (400 lines duplicated)
# After: single tests/helpers.py, imported everywhere
from tests.helpers import FakeMonitor, patch_start_monitor
```

## Checklist

When writing or reviewing a mock:
- [ ] Does it raise on invalid input like production would?
- [ ] Does every public method have the same failure modes?
- [ ] Does it handle partial operations (not just all-or-nothing)?
- [ ] Are timing primitives only patched in scoped contexts?
- [ ] Is this mock used in 3+ files? If so, is it in a shared helper?

## Common Mistakes

- Mock returns a default value instead of raising on None/invalid input. Tests pass, production crashes.
- Adding network failure simulation to some methods but not others. Tests show the system handles outages, but only for the methods you remembered.
- Treating partial fills as full fills. Position tracking goes wrong after a partial fill because the mock removed the entire position.
