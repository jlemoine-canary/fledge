# Failing tests for <phase-id>

## Files
- <path> — <N> new tests

## Tests and their requirements
| # | Test | File | Requirement | Fails because |
|---|---|---|---|---|
| 1 | test_signup_rejects_duplicate_email | backend/myapp/accounts/tests/test_signup.py | R1 | `validate_unique_email` does not exist |
| 2 | ... | | | |

## Fixture additions (if any)
<List new fixtures/factories you had to add for tests to even run cleanly.>

## Confirmed red — <ISO timestamp>
All tests above were run and failed for the expected reason.
