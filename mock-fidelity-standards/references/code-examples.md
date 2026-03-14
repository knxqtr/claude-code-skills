# Mock Fidelity Code Examples

## Rule 1: Error Behavior Must Match Production

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

## Rule 2: Cross-Cutting Concerns Must Cover ALL Methods

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

## Rule 3: Partial Operations Are Not Full Operations

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

## Rule 4: Never Globally Mock Timing Primitives

```python
# BAD: global fixture replaces sleep for all tests
@pytest.fixture(autouse=True)
def fast_sleep():
    with patch('asyncio.sleep', return_value=None):
        yield
# Result: any background loop runs millions of iterations, 100% CPU, crash

# GOOD: scope patches to specific functions, not global background loops
```

## Rule 5: Deduplicate Test Utilities Early

```python
# Before: 23 files with identical FakeMonitor class (400 lines duplicated)
# After: single tests/helpers.py, imported everywhere
from tests.helpers import FakeMonitor, patch_start_monitor
```
