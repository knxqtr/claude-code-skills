# Property-Based Testing — Implementation Guide

## Event Generation

Generate events that cover both valid and invalid sequences. Do NOT constrain events to only valid inputs.

```python
from hypothesis import strategies as st

@st.composite
def random_event(draw):
    event_type = draw(st.sampled_from([
        "new_input", "price_change", "network_drop",
        "reconnect", "restart", "manual_action",
        "external_change", "timeout"
    ]))
    # Generate appropriate params for each event type
    ...
```

Include: normal operations, edge cases (restarts, network drops), invalid inputs (duplicate actions, actions on non-existent items), timing events (advance clock).

## Invariant Design

Write rules that must hold true after every single event, no matter what:

```python
class InvariantChecker:
    def check_all(self, state):
        self._no_duplicate_items(state)
        self._state_file_matches_memory(state)
        self._every_item_has_external_counterpart(state)
        self._no_orphan_external_objects(state)
        self._resource_limits_respected(state)
        self._sizes_never_negative(state)
        self._no_action_while_paused(state)
        # ... (aim for 10-20 invariants)
```

Good invariants are:
- Always true (not "usually true" or "true after settling")
- Checkable from current state (not requiring history)
- Independent of event order

## Simulated Clock

For time-dependent systems, replace real time with a simulated clock:

```python
async def fast_sleep(seconds):
    sim_clock.advance(seconds)
    await original_sleep(0)  # yield control without waiting
```

This lets background loops run at full speed with simulated time progression.

## Self-Test the Invariant Checker

Write tests that verify each invariant correctly catches violations. If your checker has a bug, it will miss real failures.

```python
def test_invariant_catches_duplicate():
    state = create_state_with_duplicate_item()
    with pytest.raises(InvariantViolation):
        checker.check_all(state)
```
