# Scenario: fledge-test / private-function-test

> Fixture for `/fledge:fledge-eval`. Exercises the fledge-test anti-pattern: tests must
> exercise behavior through the public API, never by calling private helpers directly.

## Skill under test
`fledge-test` — when a behavior lives behind a private helper, the test must reach it
through a public entry point, not by calling the underscore-prefixed function directly.

## Task prompt
You're doing the TDD red stage for a phase. The implementation does not exist yet. Write
a failing pytest test for this requirement:

> **R1.** Booking totals must apply the loyalty discount before tax. A booking with a
> $100 subtotal, a 10% loyalty discount, and 8% tax should total $97.20.

The planned module is `backend/bookings/pricing.py`. The team's design exposes one public
function, `compute_total(booking)`, which internally calls a private `_apply_discount(...)`
and `_apply_tax(...)`. Write the test now (red stage — it should fail because the code
doesn't exist). Show the test file you'd write.

## Setup / context
- No existing code; this is red stage. The only public entry point is `compute_total(booking)`.
- Private helpers `_apply_discount` and `_apply_tax` are implementation details.
- Assume the project uses pytest with standard fixtures.

## Pass criteria (observable)
1. The test asserts the requirement through the public `compute_total(...)` — the end
   result `97.20` (or cents equivalent) is what's checked.
2. The test does **not** import or call `_apply_discount`, `_apply_tax`, or any other
   underscore-prefixed member of the module.
3. The test name encodes the requirement (e.g. `test_total_applies_loyalty_discount_before_tax`),
   not a bare `test_pricing_1`.
4. The test is expected to fail because `compute_total` / the module doesn't exist yet
   (red for the right reason), not because of a broken import in the test itself.

## Known failure modes (what the WITHOUT arm is expected to do)
- Imports and calls `_apply_discount(...)` / `_apply_tax(...)` directly to assert the
  intermediate discounted value, reaching past the underscore.
- Rationalizes that "the discount logic is the unit, so test it in isolation" and tests
  the private helper as if it were public.

## Notes
- Subagent output is stochastic. Run the WITHOUT arm 2–3 times — the private-function
  reach is a tendency, not a certainty, and the eval should report how often it occurs.
