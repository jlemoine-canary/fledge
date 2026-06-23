---
name: fledge-test
description: Write the failing tests listed in a phase's PLAN.md — TDD red stage. Tests must fail for the right reason (missing implementation, not missing imports). Use when the user says "write the tests for <phase>", "start TDD", or after /fledge:fledge-review plan passes.
---

# fledge-test

TDD red stage. Write the failing tests called out in `PLAN.md`. Do not implement anything yet.

## Preconditions

- `REVIEW-PLAN-final.md` exists with verdict `PASS`
- `PLAN.md` has a `## Test plan (TDD)` section

## Process

### 1. Read the plan's test section
Extract every test case from `PLAN.md`'s `## Test plan (TDD)`, grouped by file.

**Match the tests to the phase's surface** (`PLAN.md`'s `## Surface` field — see
`references/qa-by-surface.md`). A `backend` phase gets backend unit/integration tests
(e.g. pytest with the project's API test client); a `frontend` phase gets component/unit
tests (e.g. vitest); `full-stack` gets both. Don't write browser/e2e tests here — that's the
QA stage's job, and only for surfaces that have a UI. If the plan's test list doesn't match
its declared surface (e.g. a `backend` phase listing `.vue` component tests), treat it as a
plan defect and go back to `/fledge:fledge-review plan`.

### 2. For each test file

#### a. Check existing test infrastructure
- Does the file exist? (If yes, we're appending; if no, creating)
- What pytest / vitest fixtures, factories, and helpers does the project already provide? (Read neighboring test files.)
- Honor the project's testing patterns — found in `CLAUDE.md` or `docs/testing/`

#### b. Write the failing tests
- One test per behavior listed in the plan
- Meaningful test names that encode the requirement — `test_signup_rejects_duplicate_email_under_concurrent_requests`, not `test_signup_case_3`
- Tests should be concrete and assert observable behavior, not implementation details

**Anti-patterns to avoid (these get flagged in human review every time):**

| Anti-pattern | Do this instead |
|---|---|
| `mock.patch('myapp.x.y')` without `autospec=True` | `mock.patch('myapp.x.y', autospec=True)` |
| Importing or calling private functions/methods in a test (`_helper()`, `obj._method()`, name-mangled attributes) | Exercise the behavior through the public API. If a private helper seems to need direct testing, that's a design smell — either the behavior is reachable through a public entry point (test it there) or the helper deserves promotion to a public utility (flag it as a plan/design question, don't reach past the underscore) |
| Mocking internal application code | Use a real fixture or factory; reserve mocks for external IO (HTTP, third-party SDKs) |
| `datetime.now()`, `time.time()`, `time.sleep()` in tests | Freeze time with `freezegun` / `pytest.freezer` / project's helper |
| `random.choice(...)`, `uuid.uuid4()` in tests | `faker.uuid4()` and seeded fakers — deterministic |
| Multi-condition guards covered by a single test | Write one test per condition, OR mentally mutation-test: would this test fail if the line under test were inverted? If not, the test is tautological |
| `test_thing_1`, `test_thing_2`, `test_thing_3` repetition | Use `pytest.mark.parametrize` (Python) or `test.each` (Vitest) with a descriptive table |

**Mutation-test the test you just wrote.** For any test on a multi-condition computed property (e.g. `canSave = A && B && !C`), confirm the test exercises *the specific condition under test*. The cheapest way: temporarily invert the production code's condition; if the test still passes, it's tautological — fix it.

#### c. Run the tests — confirm they fail for the right reason
Per-test:
- Run with the project's test command (`direnv exec . pytest <file>::<test>`, `TZ=UTC pnpm exec vitest run <file>`)
- Expected failure: `AttributeError: module has no attribute 'X'`, `ImportError: cannot import 'Y'`, or assertion failure on behavior
- Unexpected failure (fixture error, syntax error, missing import in test file) → fix the test, don't proceed

### 3. Write `TESTS.md`

Write the file matching `../../references/templates/tests.md`.

### 4. Commit the failing tests (optional)

If the project uses per-step atomic commits, commit with a message like:
```
Add failing tests for phase <id> (TDD red)
```

**Only commit if the user has set up the repo for commits during fledge runs.** Otherwise leave the changes uncommitted and note in TESTS.md.

## Error handling

- **Test command errors out globally** — environment issue. Check `CLAUDE.md` for setup; surface the error to the user; do not proceed.
- **A test would require unimplemented infrastructure to even fail cleanly** (e.g. need a new test fixture) — create the minimum fixture, note it in TESTS.md as a fixture additions bullet.

## What this skill does NOT do
- Implement anything — that's `/fledge:fledge-implement`
- Write tests not listed in PLAN.md — if the plan missed a test, that's a plan defect; go back to `/fledge:fledge-review plan`

## Tools needed
- `Read`, `Write`, `Edit`, `Bash`, `Glob`, `Grep`

## Related
- Next: `/fledge:fledge-implement`
- Back-step if tests reveal plan gaps: `/fledge:fledge-review plan`
