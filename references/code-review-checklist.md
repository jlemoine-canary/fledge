# Code & plan review checklist

Detailed items the reviewer personas consult during a review. The personas hold the *lens* (constructive, adversarial, integrator) — this file holds the *items*.

Read this whole file at the start of a review. Don't rely on the one-liners in the persona alone.

---

## Plan-mode hunting items

### P1. Silent source-of-truth resolutions
If the SoT is ambiguous on a consequential point and the plan picked a direction without documenting it under `Open questions`, that's a finding.

The highest-cost shape of this is **"add a new X" vs "change the existing X"** — a new merge tag vs. redefining the existing one, a new field vs. widening the current one, a new endpoint vs. extending the old one. The two readings produce working code either way, so nothing downstream catches it; it surfaces weeks later as a revert. When the SoT admits both readings, that is always an `Open questions` entry, never a silent pick.

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

### P10. Data-migration completeness
P4 covers migration *ordering* and gating. This item covers the rows and queries that already exist. For any new column, filter, or index, answer all three:

- **Who owns the existing rows?** A column added `DEFAULT false` plus a filter that excludes false rows silently hides every pre-existing row. If the plan adds a filter, it must name the backfill that populates history before the filter goes live — or state why history legitimately belongs on the excluded side.
- **Which *other* query's plan just changed?** Narrowing an index to a partial index removes it from every consumer whose predicate no longer matches; that query is then free to seq-scan a large table. Enumerate the existing consumers of any index being replaced or narrowed.
- **Which sibling surfaces share the model?** A one-time remediation scoped to one surface leaves the sibling surfaces using the same model in the broken state. Either cover them or explicitly defer them in writing — don't let "complete" mean "complete for the surface I was looking at."

Also check that a remediation's `UPDATE`/`DELETE` re-applies the same predicates used to *select* the candidate ids. Re-reading ids into a bare `filter(id__in=...)` drops the safety bounds the selection step established, and rows can change between the two steps.

---

## Code-mode hunting items

### C1. Test quality
- Tests that import or call private functions/methods directly (`_helper()`, `obj._method()`, name-mangled attributes) instead of exercising the public API. Tests coupled to internals break on refactors that don't change behavior and pass when the public contract is broken. If the private helper genuinely needs its own tests, the fix is a design change (promote it to a public utility), not a test that reaches past the underscore.
- Patches without `autospec=True`
- Mocking internal application code where a real fixture would work
- `datetime.now()` / `time.time()` / `time.sleep()` instead of frozen clock
- `random.choice(...)` / `uuid.uuid4()` instead of `faker.uuid4()` and seeded fakers
- Multi-condition guards where another condition independently produces the same observable result (tautological tests). Mutation-test mentally: would this test fail if the line under test were inverted? If not, the test is bad.
- **Tests that prove the framework, not the guarantee.** The mental mutation test above is routinely applied to business logic and routinely skipped on three shapes where it matters most. For each, name what you would delete and confirm the test goes red:
  - **Database-level defaults.** A test that calls `Model.objects.create()` supplies the field value from the Python-level `default=`, so it passes with the migration's `db_default` / raw-SQL `DEFAULT` removed entirely. If the claim is "the database supplies this," the test must insert while *omitting the column* (raw SQL or a `.objects.raw()` insert) or inspect the column default in the catalog.
  - **CLI / management-command flag matrices.** A command with `--channel email|sms|all` tested only on the default branch will pass with the other branches wired wrong. Every branch of every flag that selects *what gets written* needs its own case.
  - **Arguments advertised as safety boundaries.** `--hotel-id`, `--created-before`, `--dry-run`: it is not enough to assert that omitting the flag raises. Prove **isolation** — set up two tenants, run scoped to one, assert the other is untouched. And when a test asserts an error is raised, assert *which* error (`match=`, or satisfy every other precondition), or it will keep passing for the wrong reason after the code moves.

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

New write endpoints need one extra pass: **does this reuse an auth/permission class scoped for reading?** A read-scoped auth class attached to a route that sends, charges, or dispatches grants the send to everyone who can view. Reusing the class is a finding unless the PR says why read access is the correct bar for this write.

### C10. Comment altitude
Comments should carry non-obvious constraints. They should not carry the diff, the changelog, or the reasoning that belongs in the PR description. All three are findings:

- **Comments that restate the code.** If the line below says what the comment says, delete the comment.
- **Comments that narrate the change.** "the deleted `ChannelTabsRow` used to own this", "updated to fix the null case", "moved here from X" — this is commit-message and PR-description content. In the source it goes stale the moment anything moves, and it reads as history to someone who never saw the before state.
- **Comments carrying design rationale at essay length.** Good PR-description material, wrong home. Keep the one sentence naming the constraint; move the rest.

Keep: why a non-obvious bound was chosen, which invariant makes an apparently-unsafe line safe, a link to the ticket for a deliberate deviation.

### C11. User-facing copy follows behaviour
When a matching rule, channel, condition, or unit changes, the strings describing it usually don't — nothing type-checks prose. Whenever the diff changes *what* something matches on or *when* something fires, grep the touched feature for:

- tooltips, labels, empty states, toasts, `aria-label`s
- i18n keys and their default values
- docstrings and type-hint comments on the changed function
- the PR description itself, against the final diff

A feature that now matches on email address while its tooltip still reads "Automatically linked via matching phone number" is a user-visible defect, not a nit.

### C12. YAGNI — machinery without a caller
For every new abstraction, cache, memo, parameter, hook, or config knob: **does it have a caller in this diff?**

If not, it is a finding unless the plan names the specific consumer and the phase that lands it. "We'll need it when X" is not sufficient — without a real access pattern the design is a guess, and the complexity is paid now while the benefit is hypothetical. Memoization is the most common offender: it adds a scope, an invalidation question, and a correctness surface, and it cannot be tuned against a usage pattern that doesn't exist yet.

Related shapes to flag:
- A function whose body has several distinct responsibilities that could be read independently (load candidates / resolve / log exclusions). Ask for the seam rather than trying to follow it.
- A micro-optimization that hand-assigns what the ORM would fetch, with nothing keeping it honest if the query above it changes. Either drop it for the obvious `select_related`, or make it self-validating (assert the ids match, raise if not).
- Two mechanisms enforcing the same condition (a `v-if` and an `enabled-for-*` prop with identical predicates). One is dead; find out which and delete it.
- A test suite far larger than the code it covers — see C1, but also ask whether the *feature* is oversized rather than the tests being thorough.

### C13. Write-endpoint idempotency
Every new `POST`/`PUT` that causes an external side effect — sends a message, charges, dispatches a task, writes to a third party — needs an answer to "what happens on a double-click or a client retry?" No answer is a finding.

When a dedup lock or cache key is the answer, check the lifecycle:
- **Acquire after cheap validation, not before.** A lock taken ahead of the prerequisite checks stays held when validation raises, so the legitimate retry hits the duplicate path.
- **Release on every failure path.** `cache.add(key)` followed by a `.delay()` that throws leaves the key set for its full TTL, suppressing every subsequent attempt until it expires.
- **Return the row the caller asked about.** A dedup path keyed on a body hash that returns "the most recent send on the thread" returns the wrong record when two different bodies interleave.
- **Partial-failure cleanup.** Two sequential writes to an object store, or a DB write paired with an external send, need to say what happens when the second one fails and the first already landed.

---

## Doc-concerns reporting

If a doc you reviewed against seems wrong, outdated, or contradicts the source-of-truth, raise it as a `Doc concerns` finding in your review. Don't apply or skip a rule silently — surface stale docs so the user can update.
