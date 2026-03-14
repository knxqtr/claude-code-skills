---
name: property-based-testing
description: Property-based fuzz testing with Hypothesis — random event sequences and invariant checking for state machines. Use when testing complex stateful systems, event-driven logic, or when hand-written tests miss edge cases. Use when user says "fuzz test", "property test", "Hypothesis", "invariant", "random testing", "simulation", or "state machine test".
---

# Property-Based Testing

## When to Use

- System has complex state that changes based on sequences of events
- Hand-written tests only cover scenarios you thought of
- You need to find bugs in event orderings humans would not write
- Testing state machines, workflows, or event-driven architectures

## Core Concept

Instead of writing specific test cases, define:
1. How to generate random events
2. What must always be true (invariants)

The framework generates thousands of random event sequences and checks invariants after every event. When it finds a failure, it automatically shrinks the sequence to the minimal reproduction.

## Architecture

```
tests/sim_harness/
  generator.py         # Hypothesis strategies for random events
  smart_mock.py        # Realistic simulator (exchange, database, etc.)
  invariants.py        # Rules checked after every event
  test_simulation.py   # Harness runner
  test_invariant_checker.py  # Self-tests for the invariant checker
```

## Event Generation

Generate events that cover both valid and invalid sequences. Do NOT constrain events to only valid inputs. The system must handle everything gracefully.

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

## How a Run Works

```
1. Create simulator + system under test
2. Hypothesis generates random list of 5-100 events
3. For each event:
   a. Execute the event
   b. Drain all pending async tasks
   c. Run all invariant checks
   d. If any invariant fails → Hypothesis shrinks to minimal sequence
4. Run 1000+ sequences (more in CI)
```

## What It Finds

Bugs that hand-written tests miss:
- State corruption when two events happen in a specific order
- Recovery failures after particular event sequences
- Stuck state (item blocked forever after a specific sequence)
- Orphan resources (external objects without matching internal tracking)
- Lifecycle issues (monitors not stopped, cleanup not triggered)

## Common Mistakes

- Constraining events to only valid sequences. The system must handle invalid input gracefully. Let the generator throw everything at it.
- Writing invariants that are "usually true" with exceptions. Invariants must hold unconditionally after every event.
- Not self-testing the invariant checker. A buggy checker gives false confidence.
- Running too few sequences. Use at least 1000 in CI. More is better.
