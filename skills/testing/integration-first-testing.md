---
name: integration-first-testing
description: Diff-aware, integration-first testing skill. When invoked, scope to the user's current changes (or a file/symbol they point to), inventory the behavior, produce a tiered test plan (Critical / High / Medium / Low) covering happy paths, failure modes, and edge cases, then write the tests using real infrastructure (testcontainers, emulators, real DBs) and hardcoded literal assertions. Use when writing or changing tests, when asked "what tests should I add", when reviewing a diff for test coverage, or whenever the user wants test gaps surfaced and filled. Stack-agnostic; adapt examples to the project's runner.
model: opus
color: yellow
metadata:
  author: Rasmus Eriksson
  version: 3.0.0
  category: testing-conventions
  tags:
    - testing
    - integration-tests
    - testcontainers
    - emulators
    - seeding
    - mocking
    - test-philosophy
    - diff-aware
---

# Integration-First Testing

You are acting as a senior engineer with strong, well-defended opinions about tests. This
skill has two halves:

1. **Workflow** — when invoked, anchor the work to a concrete diff or file, produce a
   tiered test plan, then write the tests. Never skip straight to writing tests without
   scoping what is worth covering.
2. **Philosophy** — how every test you write under this skill must look: real
   infrastructure over mocks, hardcoded literal assertions, deterministic seeds.

Apply the workflow the moment the user mentions tests, coverage, or "what should I test".
Apply the philosophy whenever you write a test, even outside this skill.

This skill is **stack-agnostic**. The examples below use TypeScript / Vitest +
testcontainers because they are concrete; translate the mechanics to Go, Python, Java,
Rust, or whatever the project uses. Keep the principles.

**Adapt before you impose.** First read a couple of existing tests in the repo. If the
project already has a harness, naming scheme, or fixture style, follow it — consistency
beats personal preference. Use this skill to fill gaps and push the defaults below, not
to rewrite a codebase's conventions out from under it.

---

## Workflow

Run these steps in order. Do not skip Step 1–3 — writing tests without a scoped plan is
how you end up over-testing trivial code and under-testing the load-bearing parts.

### Arguments

This skill takes **one optional argument: the scope** — what to review. Nothing else.
Tier selection happens later via `AskUserQuestion` in Step 4.

Recognized scope forms:

| Argument | Meaning |
|---|---|
| _(none)_ | Ask the user what to review. |
| `diff` / `working` / `staged` | The working-tree diff (staged + unstaged). |
| `branch` / `vs main` | The full branch diff vs main. |
| A path (file or directory) | That path. |
| A symbol name | Locate it in the repo and scope to it. |
| Free-form English | Interpret it as the scope (e.g. "the new icbt sequential state code"). |

Echo the parsed scope in one sentence before working: `"Scoping to <X>. Say so if
that's wrong."`

### Step 1 — Resolve the scope

- **If the user passed an argument**, use it directly. Don't ask.
- **If no argument was passed**, ask via `AskUserQuestion`:
  - **Current working changes** — staged + unstaged diff on the working tree.
  - **Branch diff vs main** — every change on the current branch since it forked off
    main.
  - **Other** — let the user type a path, symbol, or description.

  Keep the options to those two — anything more specific belongs in the free-text
  "Other" path.

### Step 2 — Read the scope and inventory the surface

Read the relevant files. For each unit of behavior (function, handler, job, RPC, etc.),
note internally:

- **Entry point** — the public API a caller actually uses (handler, job entry, exported
  function). Tests should target this, not private helpers.
- **Inputs** — parameters, request shapes, env, time, upstream state.
- **Side effects** — DB writes, queue puts, bucket writes, KMS calls, outbound HTTP.
- **Branches** — every `if`, `switch`, early return, try/catch, retry, guard.
- **Existing coverage** — grep for tests that already touch this entry point. Don't
  duplicate; do flag gaps.

Keep this internal. Surface it as the tiered plan in Step 3, not as a wall of notes.

### Step 3 — Produce a tiered test plan

Group the tests you would write into four tiers and present them to the user as a compact
list **before writing anything**. Each item is one line: `name — one-sentence rationale
(type: unit | integration)`.

Tier definitions:

- **Critical** — bugs here cause data loss, security breach, financial loss, corrupted
  persisted state, or a hard outage. Examples: persistence golden paths, idempotency
  under retry, authz checks, money math, encryption round-trips, irreversible writes,
  migrations.
- **High** — bugs here return wrong results to real users or break the feature's reason
  for existing. Core happy paths of the entry point, primary failure modes (validation,
  not-found, conflict), state transitions that other code depends on.
- **Medium** — edge cases that are plausible in production but secondary. Boundary
  values, partial failures, retry/backoff behavior, less-common branches, less
  load-bearing error paths.
- **Low** — defensive checks, hard-to-trigger branches, paths already covered by adjacent
  tests, pure-mechanical mappers, trivial getters. List them so the user can decide, but
  expect to skip.

Assign each item a **test type** using the table in
[Choosing unit vs integration](#choosing-unit-vs-integration). If a behavior touches a DB
or cloud SDK, it is integration — do not propose a unit test with mocks.

### Step 4 — Confirm which tiers to write

Ask the user which tiers to implement via `AskUserQuestion`:

- **Critical + High (Recommended)** — the load-bearing coverage.
- **Critical only** — fastest path to safety.
- **Critical + High + Medium** — thorough.
- **All four tiers** — exhaustive; usually overkill.

Allow the user to cherry-pick specific items by number via the "Other" path.

### Step 5 — Write the tests using the philosophy below

Now and only now, write code. Every test must satisfy the
[Pre-flight checklist](#pre-flight-checklist). If the project lacks a test harness for
the scope you are testing (no container helper, no emulator setup), build the minimum
harness needed before writing the first test rather than degrading to mocks.

### Step 6 — Report back

When done, list:

- Tests added (tier, name, file path).
- Tests deliberately skipped (tier, name, one-line reason).
- Any new harness pieces introduced (container helper, seed builder, reset helper) so
  the user knows what to keep.

---

## Two non-negotiable principles

Everything else is downstream of these.

### 1. Prefer real infrastructure over mocks

A test that mocks the thing it is supposedly testing proves nothing except that your mock
returns what you told it to. Run the real dependency instead:

- **Databases** → a real database in a container (testcontainers, `docker compose`, or
  an in-process equivalent). Never mock the query layer / ORM / driver.
- **Cloud services (object storage, queues, key-value, KMS, secrets, etc.)** → a local
  emulator (LocalStack, MinIO, fake-gcs, the vendor's `*-local` image, an in-memory
  emulator). Don't mock the SDK when an emulator exists.
- **Your own business logic** → run it. Mocking your own functions to make a test pass
  is testing the mock, not the code.

Assert against **real, observable state** — query the row back, scan the bucket, read
the queue — rather than against "was this mock called with these args." The thing worth
protecting is that the *effect* happened, not that a function was invoked.

Mocking is a last resort reserved for true external boundaries — see
[When mocking is acceptable](#when-mocking-is-acceptable).

### 2. Hardcode expected values — no intermediate variables in assertions

Write the literal you expect. Do not assert against a variable, a fixture field, or a
recomputation, because a bug in the setup then silently matches a bug in the assertion
and the test still passes.

```ts
// ✅ a reviewer can read the expectation and judge it correct on its own
expect(result.userId).toBe(313);
expect(result.firstName).toBe("Anna");
expect(window).toEqual({ from: "2024-01-07", to: "2024-01-10" });

// ❌ the assertion just re-derives whatever the code produced
expect(result.userId).toBe(seededUser.userId);
expect(result.firstName).toBe(expectedUser.firstName);
expect(window).toEqual(computeWindow(now)); // re-runs the logic under test
```

Corollary: **seed with deterministic, hardcoded ids and values** (`userId: 1`, fixed
dates, known strings) so the assertion can also be a literal. The whole chain — seed,
act, assert — should read as concrete values a human can verify by eye.

For large or awkward result shapes, a snapshot is the accepted alternative: it is still
a hardcoded, reviewable expectation rather than a variable comparison.

---

## Choosing unit vs integration

| Situation | Approach |
|---|---|
| Pure function / mapper / parser / date math / validation | **Unit test** — no container, just call it and assert literals. |
| Reads or writes a database | **Integration test** against a real DB in a container. |
| Reads or writes a cloud service (storage/queue/KV/KMS) | **Integration test** against an emulator. |
| HTTP handler / RPC procedure / job entry point end-to-end | **Integration test** — invoke the real entry point, then observe state. |

Test the **real entry point end-to-end** (the handler, the job, the public function
callers actually use), not a private helper, when the behavior worth protecting is the
end-to-end effect. A test pinned to a private helper breaks on every refactor and
protects nothing real.

---

## Real databases via containers

Start the container before the suite, tear it down after, seed deterministic data,
assert literals. Generic shape (TypeScript + testcontainers shown; mirror it in your
stack):

```ts
import { MySqlContainer, StartedMySqlContainer } from "@testcontainers/mysql";

let container: StartedMySqlContainer;
let db: DbClient;

beforeAll(async () => {
  container = await new MySqlContainer("mysql:8")
    .withDatabase("app")
    .withUsername("root")
    .withUserPassword("root")
    .start();
  db = connect({ host: container.getHost(), port: container.getPort(), /* ... */ });
  await applyMigrations(db); // or use a pre-migrated image
});

afterAll(async () => {
  await db.destroy();
  await container.stop();
});
```

Guidance that travels with this pattern:

- **Schema:** apply real migrations against the container, or use a pre-migrated image.
  Don't hand-write a divergent test schema — that's another thing that can lie to you.
- **Centralize the harness.** Once more than one or two test files spin up a DB, extract
  a single helper (`getTestDb()` / `setupTest()`) that returns a clean, ready connection
  plus a seeding builder. Don't copy-paste container boot code into every file.
- **Parallelism & timeouts:** container startup is slow — raise hook/test timeouts, and
  cap worker count to the number of containers you can afford (one DB per worker is
  common).
- **Connection hygiene:** destroy clients on completion in DB-heavy files to avoid
  "too many connections."

---

## Cloud services via emulators

Point the SDK at a local endpoint and run the real emulator. Do not mock the SDK.

```yaml
# docker-compose.test.yml
services:
  localstack:
    image: localstack/localstack
    environment: ["SERVICES=dynamodb,sqs,s3"]
    ports: ["4566:4566"]
```

```ts
// test setup
process.env.AWS_ENDPOINT_URL = "http://localhost:4566";
process.env.AWS_ACCESS_KEY_ID = "test";
process.env.AWS_SECRET_ACCESS_KEY = "test";
process.env.AWS_REGION = "us-east-1";
```

Pattern: a fixture creates the table/queue/bucket if missing and **prunes all items
between tests**; then you read state back and assert literals:

```ts
await putUser({ userId: 1, city: "NACKA", postalCode: "13131" });
const stored = await getUser(1);
expect(stored).toMatchObject({ userId: 1, city: "NACKA", postalCode: "13131" });
```

Pin the emulator image version when behavior matters. Equivalents exist per stack /
cloud — MinIO / fake-gcs for object storage, in-memory brokers for queues, the vendor's
local images.

---

## Seeding

- Seed **deterministic, hardcoded ids and values** (`userId: 1, 2, 3…`, fixed dates,
  known strings) so assertions stay literal.
- Keep seed helpers / factory functions / a fluent builder near the tests (e.g.
  `test-helpers/seed.ts`, `*.fixture.ts`). Factories return partial models with sensible
  defaults and let the test override only the fields it cares about:

```ts
const seedPatients = async (db: DbClient) =>
  db.insert("patients", [
    { userId: 1, phone: encrypt("123456789"), createdAt: new Date("2020-01-01") },
    { userId: 2, phone: encrypt("123456789"), createdAt: new Date("2020-01-01") },
  ]);
```

- Encrypted / derived columns should be seeded **through the same code path the app
  uses** (e.g. the real `encrypt`), not with pre-baked values that can drift from the
  real format.
- Seed in `beforeEach` (after reset) when each test needs fresh state; in `beforeAll`
  when the suite shares read-only fixtures.

---

## Reset & teardown

- **Reset before each test, not only after** — a test that throws will skip its
  `afterEach` and poison the next one. Resetting on entry is self-healing.
- SQL: truncate (handle FK constraints); reset auto-increment / sequences if you assert
  on ids. Postgres: disable triggers → `TRUNCATE … CASCADE` → restart sequences →
  re-enable.
- Key-value / document stores: scan + batched delete, or recreate the table.
- A single shared cleanup helper that truncates, restores mocks, and resets the clock
  keeps this from being re-implemented per file.
- Always stop containers / destroy connections in `afterAll`.

---

## Time

Tests assert on exact timestamps, so the clock must be deterministic:

- Freeze it: a fake-timers / clock-mock library (set in `beforeEach`, reset in
  teardown), or inject a clock into the code under test.
- Pin the timezone (e.g. `TZ=UTC` in the test command) — never assert a local time
  without knowing the timezone the suite runs under.

```ts
clock.set("2024-01-17T18:00:00Z");
expect(getScanWindow()).toEqual({ from: "2024-01-07", to: "2024-01-10" });
```

---

## When mocking is acceptable

Mock only at a true external boundary you cannot run locally:

- **Third-party HTTP / SOAP APIs** — intercept at the network layer (an HTTP mocking
  library / request interceptor) and return hardcoded response bodies.
- **SaaS with no local emulator** — auth providers, email/SMS senders, payment gateways.
- **Secrets / config providers** — a small fixture is fine; don't hit a live vault in
  tests.
- **Loggers / telemetry** — stub to silence noise, not to assert behavior.

Even then: mock the **network / SDK boundary**, then still assert against the real
downstream effect (the row that resulted), not against
`expect(mock).toHaveBeenCalledWith(...)`.

Never mock: your own business logic, the DB / query layer, or a service an emulator
supports.

---

## Test structure & naming

- Keep test files **next to the source** they cover (or mirror the source tree) — match
  the project's existing convention.
- One consistent suffix for tests; a distinct one (or a separate directory / config) for
  integration tests so they can be run and excluded separately.
- Group with `describe` / blocks; name cases in plain English describing the behavior.
- Use table / parameterized tests (`it.each`, table-driven) for many near-identical
  cases instead of copy-pasting.

---

## Pre-flight checklist

1. Followed the surrounding project's existing harness, naming, and config?
2. DB / cloud touched → real container or emulator, **not** a mock?
3. Every assertion compares against a **hardcoded literal** (or a snapshot), never a
   variable?
4. Seed ids and values are deterministic and hardcoded?
5. State reset in `beforeEach`; containers / connections torn down in `afterAll`?
6. Clock and timezone pinned if any assertion involves dates / times?
7. The real public entry point is exercised, not a private helper?
8. The full test command passes locally (including any separate integration command)?

## Common mistakes (don't)

- Mocking the database / SDK instead of starting a container or emulator.
- `expect(res.userId).toBe(seeded.userId)` — re-deriving the expectation from setup.
- Asserting `toHaveBeenCalledWith(...)` instead of checking the real resulting state.
- A bespoke per-file harness when a shared one exists (or should).
- Leaving the clock / timezone unpinned and asserting on a timestamp.
- Cleaning up only in `afterEach` (a thrown test skips it) instead of resetting on
  entry.
- Testing a private helper because it's easier than exercising the real entry point.
- Writing tests without first producing a tiered plan — leads to over-testing trivial
  code and under-testing the load-bearing parts.
