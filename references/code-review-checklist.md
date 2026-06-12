# Code & plan review checklist

Detailed items the reviewer personas consult during a review. The personas hold the *lens* (constructive, adversarial, integrator) — this file holds the *items*.

Read this whole file at the start of a review. Don't rely on the one-liners in the persona alone.

---

## Plan-mode hunting items

### P1. Silent source-of-truth resolutions
If the SoT is ambiguous on a consequential point and the plan picked a direction without documenting it under `Open questions`, that's a finding.

### P2. Coverage gaps masked by wording
Requirements may look covered because phrasing overlaps, even though the plan doesn't actually address them. Trace each requirement to a specific plan item; mismatches are findings.

### P3. Sub-phase implementation knots
Sub-phases that share state or invariants with no explicit contract will produce integration pain at implement time. Look for unstated shared assumptions between siblings.

### P4. Migration / rollout / feature-flag gating
Does the plan touch a high-volume path (PMS integrations, message scheduling, payments, anything `CLAUDE.md` flags as high-risk) without naming a feature flag, dynamic-rollout gate, or migration ordering? If yes, finding.

### P5. Bounded-context coupling
Look for *new* cross-app imports that don't go through a shared location or follow established precedent. Imports from shared utilities or following existing patterns are fine. Continuing a known-bad pattern when a better one is in reach is also a finding.

### P6. Reuse vs novelty under-justified
Spot-check the `Reuse vs novelty` table. Was reuse genuinely a fit, or was it dodged? Is novelty justified by a concrete defect/mismatch — not just style?

### P7. Rollback gaps
Every risk in `Risks` should have a rollback path. Missing rollback for a risky change is a finding. Migrations should be reversible (or have a documented forward-only justification).

### P8. Second-order effects
- Performance under load (the plan looks fine at 1 RPS; what about 1000?)
- Cache staleness / invalidation paths
- Index bloat, query plan changes
- Connection pool starvation
- Reversibility under partial-failure

### P9. Security
Run the full security section of the project `CLAUDE.md`, line by line, against the plan. Common: auth bypass via happy path, PII leakage into logs, secret exposure, injection surface, broken CSRF, broad CORS.

---

## Code-mode hunting items

### C1. Test quality
- Tests that import or call private functions/methods directly (`_helper()`, `obj._method()`, name-mangled attributes) instead of exercising the public API. Tests coupled to internals break on refactors that don't change behavior and pass when the public contract is broken. If the private helper genuinely needs its own tests, the fix is a design change (promote it to a public utility), not a test that reaches past the underscore.
- Patches without `autospec=True`
- Mocking internal application code where a real fixture would work
- `datetime.now()` / `time.time()` / `time.sleep()` instead of frozen clock
- `random.choice(...)` / `uuid.uuid4()` instead of `faker.uuid4()` and seeded fakers
- Multi-condition guards where another condition independently produces the same observable result (tautological tests). Mutation-test mentally: would this test fail if the line under test were inverted? If not, the test is bad.

### C2. Logging discipline
- `logger.error(...)` / `logger.warning(...)` inside business logic that doesn't `bound_contextvars(hotel=..., reservation=..., correlation_id=...)`. Especially hotel-scoped or vendor-scoped operations.
- Silent error swallowing — `except Exception: pass`, broad except without re-raise or explicit logging, decorators that catch and continue
- Logging that would page oncall at 3am for a benign event (use `logger.info` or drop entirely)

### C3. Naming, constants, enums
- String literals that should be constants
- Integer counts that should be named (`days=1` → `CHECK_IN_ACCESS_GRACE_PERIOD_DAYS`)
- Fixed string sets that should be `StrEnum` / `Literal`
- Function names that don't match behavior (e.g. `get_X` that also creates side effects, `process_Y` that filters)

### C4. Validation at the wrong layer
Runtime checks that should be schema-level. Prefer `Literal[...]`, `Annotated[str, Meta(min_length=1, max_length=255)]` via msgspec. Boundary validation gives clearer errors than runtime asserts and protects deserialization.

### C5. Concurrency / race conditions
- Every "check then create" pattern needs `select_for_update()` OR a unique constraint at the DB
- Cross-store writes (e.g. Postgres + OpenSearch / DynamoDB) need explicit handling for the second-write failure
- Soft-delete semantics consistent with the rest of the codebase
- **Multi-tenant filter completeness**: enumerate every tenant-bearing FK reachable through the query's joins (a `Message` has both `thread.hotel` and `scheduled_message.hotel`; a Reservation has both `hotel` and `guest.hotel`). If only one is filtered, that's a cross-tenant leak path even without a known invariant that the two are equal. Single-FK filtering is a finding unless an enforced invariant (DB constraint, signal) is cited.

### C6. Exception handling
- `assert` in production code → use `AppException` or domain exception
- Broad `except` without strong rationale
- Variably-described exceptions that Sentry can't group — make `vendor` (or whatever varies) a kwarg, not part of the message string

### C7. Boundary correctness
- Off-by-one, null/empty/oversized input
- Unicode edge cases (combining characters, RTL, normalization)
- Timezone confusion (naive vs aware, UTC drift)
- N+1 queries, missing indexes, unbounded result sets
- **Django ORM efficiency smells**:
  - `.distinct()` on a queryset is almost always a sign of JOIN multiplication through a one-to-many or reverse-GenericRelation. `count()` becomes `COUNT(DISTINCT ...)` over the multiplied rowset. Prefer `Exists()` / `Subquery` to test "has-a-related" without joining.
  - `.exclude() / .order_by() / .filter()` called on a relation manager that's already in a sibling `prefetch_related()` invalidates the prefetch cache and re-queries. Walk the in-memory list (`sorted(rel.all(), ...)`, `next(e for e in rel.all() if ...)`) instead — keeps both call sites consistent and saves a round-trip.
  - Joining through reverse one-to-many relations (`parent__children__field`) without thinking about how many rows the join produces.

### C8. Cleanup gaps
- Stale LLM comments ("// updated to fix bug", openspec recommendations, "removed X" markers)
- Leftover screenshots, debug print statements
- IMPLEMENTATION.md / PR description doesn't match the actual diff (Copilot catches this constantly — get there first)
- **Defensive code for scenarios the type system / queryset / model invariants make unreachable.** If the project `CLAUDE.md` says "trust framework guarantees, only validate at system boundaries" (most do), guards like `if foo is None: return None` for a foo whose queryset filter excludes None are dead code — coverage CI will reject them, and they invite future maintainers to think the case can happen. Reviewers (especially the adversarial pass) should **NOT** recommend adding such guards. If you must, also require a test that exercises the guard — otherwise drop it. Pyright-narrowing `assert foo is not None` lines are fine because they execute on every call.

### C9. Security
Run the full security section of the project `CLAUDE.md`, line by line, against the diff. Common: input validation gaps, SQL/command injection surface, v-html on user content, PII in logs, secrets in source.

---

## Doc-concerns reporting

If a doc you reviewed against seems wrong, outdated, or contradicts the source-of-truth, raise it as a `Doc concerns` finding in your review. Don't apply or skip a rule silently — surface stale docs so the user can update.
