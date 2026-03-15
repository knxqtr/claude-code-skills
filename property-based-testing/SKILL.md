---
name: property-based-testing
description: Property-based fuzz testing with Hypothesis for stateful systems. Use when hand-written tests miss edge cases, testing event-driven architectures, or verifying invariants across random event sequences. Covers event generation, invariant design, and simulated clocks. Use when user says "fuzz test", "random testing", "find edge cases", or "state machine testing".
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

## How a Run Works

1. Create simulator + system under test
2. Hypothesis generates random list of 5-100 events
3. For each event: execute it, drain all pending async tasks, run all invariant checks
4. If any invariant fails, Hypothesis shrinks to the minimal sequence
5. Run 1000+ sequences (more in CI)

## What It Finds

Bugs that hand-written tests miss:
- State corruption when two events happen in a specific order
- Recovery failures after particular event sequences
- Stuck state (item blocked forever after a specific sequence)
- Orphan resources (external objects without matching internal tracking)
- Lifecycle issues (monitors not stopped, cleanup not triggered)

## Key Design Rules

- Generate both valid and invalid events. Do NOT constrain to only valid inputs. The system must handle everything gracefully.
- Invariants must hold unconditionally after every single event. Not "usually true" or "true after settling."
- Invariants must be checkable from current state alone, not requiring history.
- Always self-test the invariant checker -- a buggy checker gives false confidence.
- For time-dependent systems, use a simulated clock instead of real time.

## Common Mistakes

- Constraining events to only valid sequences. The system must handle invalid input gracefully.
- Writing invariants that are "usually true" with exceptions. Invariants must hold unconditionally.
- Not self-testing the invariant checker. A buggy checker gives false confidence.
- Running too few sequences. Use at least 1000 in CI. More is better.

For implementation details, see references/implementation-guide.md in this skill's directory.
