---
name: mock-fidelity-standards
description: Test mock quality standards ensuring mocks reproduce real error behavior. Use when writing mocks, fakes, stubs, or test simulators. Covers error fidelity, cross-cutting failure modes, partial operations, and timing mock safety. Use when user says "tests pass but production fails", "mock too lenient", or "flaky tests".
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

1. Error behavior must match production. If production raises on invalid input, the mock must raise too.
2. Cross-cutting concerns must cover ALL methods. When adding a failure mode (network sim, rate limiting) to a mock, audit every public method. Missing one creates a hole where tests pass but production fails.
3. Partial operations are not full operations. If the real system can partially complete (partial fill, partial delete), the mock must model this. "Reduce by X" and "remove entirely" are fundamentally different.
4. Never globally mock timing primitives. Replacing asyncio.sleep globally makes background loops spin at infinite speed, causing 100% CPU and crashes. Scope patches to specific functions.
5. Deduplicate test utilities early. When 3+ test files share the same mock class, extract to a shared helper immediately. Duplicated mocks drift apart over time.
6. Test data factories must use the same types as the real models. If a Pydantic model defines `extra_args: dict[str, str]`, the test helper must pass a dict, not an empty string. A factory that uses the wrong type tests a code path that production never hits.

For code examples, see references/code-examples.md in this skill's directory.

## Checklist

When writing or reviewing a mock:
- [ ] Does it raise on invalid input like production would?
- [ ] Does every public method have the same failure modes?
- [ ] Does it handle partial operations (not just all-or-nothing)?
- [ ] Are timing primitives only patched in scoped contexts?
- [ ] Is this mock used in 3+ files? If so, is it in a shared helper?
- [ ] Do test data factories use the same types as the real models?

## Common Mistakes

- Mock returns a default value instead of raising on None/invalid input. Tests pass, production crashes.
- Adding network failure simulation to some methods but not others. Tests show the system handles outages, but only for the methods you remembered.
- Treating partial fills as full fills. Position tracking goes wrong after a partial fill because the mock removed the entire position.
