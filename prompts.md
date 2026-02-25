# Adaptive Coding Coach — TDD Code-Generation Prompts (from `plan.md`)

Owner: Camille Grieve  
Date: 2026-02-25

These prompts are designed to be fed **sequentially** into a code-generation LLM.  
They enforce **test-driven development**, small increments, and always end with code fully wired into the running app (no orphaned modules).

---

## Prompt 0 — Ground rules + repo context (run once)

```text
You are an expert full-stack engineer. We are implementing the MVP described in spec.md and the step plan in plan.md. Use test-driven development: write or update tests first, then implement. Keep each change small. Avoid large refactors unless necessary.

Assume a monorepo with:
- backend/ (FastAPI + SQLAlchemy + Alembic + Pytest)
- frontend/ (React + TypeScript + Vite + Vitest + React Testing Library)

If the current repo differs, adapt minimally but keep the same structure. Always:
1) List files you will change
2) Add/modify tests first
3) Implement
4) Show how to run tests
5) Ensure code is wired into app routes/exports (no dead code)
Proceed with Prompt 1 now.
```

---

## Prompt 1 — Backend boot: create app + /health

```text
Step 1 (from plan.md): Backend boot.

Goal:
- Create FastAPI backend app with a /health route returning JSON { "status": "ok" } and HTTP 200.

TDD requirements:
- Add pytest test that calls /health and asserts status_code==200 and JSON payload matches exactly.

Implementation details:
- Use FastAPI TestClient.
- Ensure backend can be started locally (e.g., uvicorn backend.main:app).
- Wire /health into the FastAPI app (no placeholder code).

Please list files changed, write tests first, then implement. End by showing command(s) to run tests.
```

---

## Prompt 2 — Users table + migration + unique username

```text
Step 2: User model + migrations.

Goal:
- Add Users table/model with fields: id (pk), username (unique), password_hash, created_at.
- Use Alembic migration (or equivalent) and ensure it runs.

TDD requirements:
- Add an integration test that inserts two users with the same username and asserts the second insert fails (IntegrityError or 400 via endpoint—choose consistent approach).
- Prefer DB-level unique constraint.

Implementation:
- Introduce SQLAlchemy Base, SessionLocal, engine config.
- Use SQLite for dev/tests.
- Provide a clean test database setup/teardown.

Wire everything:
- Database must initialize at app startup.
- Tests must pass via pytest.

List files changed, tests first, then implementation, then how to run migrations and tests.
```

---

## Prompt 3 — Signup endpoint (POST /auth/signup) with validation

```text
Step 3: Signup endpoint.

Goal:
- Implement POST /auth/signup that accepts JSON: { "username": string, "password": string }.
- Validate: non-empty, reasonable length (e.g., username 3–30, password 8+).
- Hash password (bcrypt recommended) and store user.

TDD requirements:
- Tests:
  1) Successful signup returns 201 (or 200) and returns { "username": "<name>" }.
  2) Missing fields returns 422/400.
  3) Duplicate username returns 409/400.

Implementation:
- Add Pydantic schemas.
- Hash passwords securely; never store plaintext.
- Return consistent errors.

Wire:
- Add /auth router mounted in main app.

Provide files changed, tests first, implementation, and pytest command.
```

---

## Prompt 4 — Login endpoint (POST /auth/login) with cookie session

```text
Step 4: Login endpoint.

Goal:
- Implement POST /auth/login with JSON { "username": string, "password": string }.
- Verify password hash.
- Establish authentication via secure cookie session (preferred) OR JWT in HttpOnly cookie.

TDD requirements:
- Tests:
  1) Wrong password returns 401.
  2) Correct login returns 200 and sets a session cookie.
  3) Subsequent request to a protected endpoint succeeds with cookie.

Implementation guidance:
- Minimal session management: server-side session store in DB or signed cookie token is okay.
- Keep it simple and secure: HttpOnly, SameSite=Lax for local dev.

Wire:
- Ensure auth dependency/middleware is available for protected routes.

List files changed, tests first, implement, show how to run tests.
```

---

## Prompt 5 — Logout endpoint (POST /auth/logout)

```text
Step 5: Logout endpoint.

Goal:
- Implement POST /auth/logout which clears authentication (invalidates session or clears cookie).

TDD requirements:
- Test:
  1) Login, then logout, then protected endpoint returns 401.
  2) Logout when not logged in returns 200 (idempotent) or 401 (choose and test).

Wire:
- Ensure cookie cleared in response.

Proceed with tests first, then implement, then run tests.
```

---

## Prompt 6 — /me endpoint (GET /me) requires auth

```text
Step 5 (part): /me endpoint.

Goal:
- GET /me returns current user info: { "username": "...", "progress": { ...optional minimal } }
- Must require authentication.

TDD:
- Tests:
  1) Unauthenticated GET /me returns 401.
  2) Authenticated GET /me returns 200 and correct username.

Wire:
- Add route and auth dependency.

Do tests first, then implement, then provide test command.
```

---

## Prompt 7 — Levels table + seed Level 1 + GET /levels ordering

```text
Step 6: Levels content minimal.

Goal:
- Add Levels table: id, level_number, title, description.
- Seed at least Level 1 into DB on startup or via seed script.
- Implement GET /levels returning ordered by level_number.

TDD:
- Test GET /levels returns array with Level 1 and ordered properly.

Wire:
- Add content router to app.
- Ensure seed runs in dev/test in a deterministic way (avoid seeding duplicates).

Implement with tests first, then code, then how to run tests.
```

---

## Prompt 8 — Exercises table + seed one guided exercise + GET /levels/{id}/exercises

```text
Step 7: Exercises content minimal.

Goal:
- Add Exercises table with fields needed for MVP minimal exercise rendering:
  id, level_id (fk), title, prompt, scaffold_code, hints (json array), solution_code, explanation, concept_tags (json array), tests (json or text), difficulty_variant(optional).
- Seed one exercise for Level 1.
- Implement GET /levels/{id}/exercises.

TDD:
- Tests:
  1) GET /levels/{level1_id}/exercises returns seeded exercise.
  2) Nonexistent level returns 404 OR empty list (choose one and enforce consistently).

Wire:
- Add to content router.

Tests first, then implement.
```

---

## Prompt 9 — Frontend boot + routing + basic auth flow

```text
Step 8: Frontend boot + auth screens.

Goal:
- Create React+TS app (Vite) with routes:
  /login, /signup, / (dashboard placeholder), /levels.
- Implement Signup and Login forms that call backend endpoints.
- Store auth via cookie (no localStorage token).
- On app load, call GET /me to determine logged-in state.

TDD:
- Add Vitest + React Testing Library tests:
  1) Signup form renders and validates required inputs.
  2) Login form renders.
  3) Routing works (renders correct page for /login).

Wire:
- Ensure forms actually call API layer (fetch wrapper).
- Create a minimal API client module and use it in components.

Provide files changed, tests first, then implementation, and how to run frontend tests.
```

---

## Prompt 10 — Levels list UI (GET /levels) with loading/error states

```text
Step 9: Level list UI.

Goal:
- Implement /levels page that fetches GET /levels and renders list items.
- Show loading state and an error message on fetch failure.

TDD:
- Tests:
  1) Shows loading indicator initially.
  2) Renders levels after mocked API resolves.
  3) Shows error state if API rejects.

Wire:
- Use the existing API client module.
- Ensure this page is reachable from dashboard or nav.

Tests first, then implement.
```

---

## Prompt 11 — Exercise screen skeleton (no pyodide yet)

```text
Step 10: Exercise screen skeleton.

Goal:
- Create route: /levels/:levelId/exercises/:exerciseId
- Fetch exercises for a level and display selected exercise:
  prompt, scaffold editor (textarea is okay for now), Run button, Output panel.
- No pyodide integration yet; Run button can be disabled with a message "Execution engine loading" for now.

TDD:
- Tests:
  1) Renders prompt and scaffold code after load.
  2) Run button exists and is disabled (for now).

Wire:
- Link from Level list to first exercise of the level.
- Avoid orphan pages.

Tests first, then implement.
```

---

## Prompt 12 — Pyodide loader service + UI loading state

```text
Step 11: Pyodide loader.

Goal:
- Add a PyodideLoader module that loads pyodide once and caches the instance.
- Display loading state on Exercise screen until ready, then enable Run.

TDD:
- Tests:
  1) Exercise screen shows "Loading Python..." while loader promise pending.
  2) Run enabled when loader resolves (mock loader in tests).

Implementation:
- Consider loading pyodide in a Web Worker later; for now load in main thread but architect loader so worker swap is easy.

Wire:
- Exercise screen must use loader and set state.

Tests first, then implement.
```

---

## Prompt 13 — “Run code” baseline: execute and capture stdout/errors

```text
Step 12: Run code baseline.

Goal:
- Implement Run button: execute code in Pyodide and capture stdout and exceptions.
- Display output in Output panel:
  - stdout content
  - if error, show error type and message.

TDD:
- Tests (mock execution function):
  1) When execution returns stdout, UI shows it.
  2) When execution throws SyntaxError, UI shows error message.

Implementation:
- Create an execution function that returns { stdout, error }.
- Keep it deterministic and testable.

Wire:
- Hook execution function into Exercise screen Run button.

Tests first, then implement.
```

---

## Prompt 14 — Exercise evaluation harness: append asserts for pass/fail

```text
Step 13: Evaluation harness.

Goal:
- For exercises, store a test harness (simple string) in the exercise record (e.g., JSON field "tests").
- On Run, evaluate by appending harness to user code and executing; determine pass/fail.
- Show verdict: PASS / FAIL.

TDD:
- Unit tests for evaluator function (pure):
  1) correct code passes
  2) wrong code fails due to assertion

Wire:
- Exercise screen displays verdict.

Keep it minimal: one exercise harness to start.
```

---

## Prompt 15 — Mistake classification (syntax vs logic vs concept/other)

```text
Step 14: Mistake classification.

Goal:
- Implement classifier that maps execution results to mistake_type:
  - SyntaxError/IndentationError -> "syntax"
  - AssertionError -> "logic"
  - NameError/TypeError -> "concept" (or "other" if you prefer; choose one)
- Display a friendly message based on mistake_type.

TDD:
- Tests for classifier function (pure) covering all 3 cases.

Wire:
- Exercise screen shows mistake_type and message when failing.

No long-term tracking yet.
```

---

## Prompt 16 — Hint system (1→3) integrated into Exercise screen

```text
Step 15: Hints (1→3).

Goal:
- Add Hint button that reveals hints sequentially from exercise.hints array.
- Track hint_count in component state per exercise session.
- Display current hint and hint count (e.g., "Hint 2/3").

TDD:
- Tests:
  1) Click hint shows hint #1
  2) Click again shows hint #2
  3) Click again shows hint #3
  4) Further clicks keep showing hint #3 (or disables button—choose and test)

Wire:
- Ensure hints come from backend exercise payload and are shown in UI.
```

---

## Prompt 17 — Reveal solution after limit + next incorrect attempt

```text
Step 16: Reveal solution after limit.

Goal:
- After user has viewed 3 hints, if they run again and still fail, reveal:
  - exercise.explanation
  - exercise.solution_code
- Mark state as "solution revealed" and keep visible.

TDD:
- Tests:
  1) After 3 hints + failed run, solution appears.
  2) If run passes, solution should NOT appear.

Wire:
- Integrated into existing run flow.
```

---

## Prompt 18 — Attempts persistence: POST /attempts + frontend integration

```text
Step 17: Attempt persistence.

Backend:
- Create Attempts table and POST /attempts endpoint with validation:
  user_id inferred from auth, and payload includes exercise_id, submitted_code, result, hint_count, mistake_type, confidence(optional), timestamp server-side.
- Return saved attempt id.

Frontend:
- After every run, post attempt to backend.

TDD:
- Backend tests for:
  1) Auth required
  2) Valid payload creates row
  3) Invalid payload rejected
- Frontend tests:
  1) When run completes, API client is called with attempt payload (mock fetch)

Wire:
- Ensure run flow persists attempts; handle failures gracefully (show non-blocking toast/message).
```

---

## Prompt 19 — LevelProgress baseline + dashboard status

```text
Step 18: LevelProgress baseline.

Backend:
- Create LevelProgress table and ensure a row is created/updated when first attempt occurs in a level.
- Provide in GET /me minimal progress summary (current level status).

Frontend:
- Dashboard shows "Current Level" and status (locked/in_progress/passed) using /me.

TDD:
- Backend tests:
  1) First attempt creates LevelProgress in_progress
  2) /me returns progress info
- Frontend tests:
  1) Dashboard renders progress fetched from /me

Wire:
- Ensure user can navigate dashboard -> levels -> exercise.
```

---

## Prompt 20 — Quiz: multiple choice only (Level 1) + scoring endpoint

```text
Step 19: Quiz (MC only).

Backend:
- Seed quiz questions for Level 1 (MC).
- Implement GET /levels/{id}/quiz returning questions (without answers).
- Implement POST /levels/{id}/quiz-submission that accepts answers and returns score.

Frontend:
- Implement Quiz screen for MC:
  - render options
  - collect answers
  - submit
  - show score

TDD:
- Backend tests for quiz fetch and scoring.
- Frontend tests:
  1) Renders MC questions
  2) Selecting answers and submitting calls API

Wire:
- Add button at end of Level 1 exercises to "Take Quiz".
```

---

## Prompt 21 — Quiz unlock: pass threshold updates LevelProgress & unlocks Level 2

```text
Step 20: Quiz unlock.

Goal:
- Define pass threshold (e.g., 70%).
- If score >= threshold:
  - update LevelProgress.status = "passed"
  - unlock next level (e.g., LevelProgress row for next level set to unlocked/in_progress/locked logic)
- Ensure GET /levels includes lock status for the user OR /me provides enough info for UI.

TDD:
- Backend tests:
  1) Passing submission updates progress
  2) Failing submission does not
- Frontend tests:
  1) After pass, UI shows next level unlocked and provides navigation.

Wire:
- End-to-end: Level 1 quiz pass -> Level 2 accessible.
```

---

## Prompt 22 — Add guided quiz items + free response (deterministic scoring)

```text
Step 21: Add guided + free response quiz items.

Goal:
- Extend quiz to support:
  - guided (short fill-in; exact match with normalization)
  - free response (MVP deterministic scoring via keyword rubric OR exact match; choose and document)
- Update backend payload schema for quiz questions and submissions.
- Update frontend quiz renderer to handle each type.

TDD:
- Backend tests for normalization and scoring.
- Frontend tests that each question type renders and can be submitted.

Wire:
- Ensure mixed quiz works for Level 1.
```

---

## Prompt 23 — Confidence capture (Easy/OK/Hard) and store on Attempt

```text
Step 22: Confidence capture.

Frontend:
- After each exercise completion (or each attempt—choose simplest), show a prompt: Easy / OK / Hard.
- Include selected confidence in attempt POST.

Backend:
- Validate confidence enum and store it.

TDD:
- Frontend tests:
  1) Confidence selector appears after run result
  2) Attempt payload includes confidence when selected
- Backend tests:
  1) Accepts valid confidence
  2) Rejects invalid value

Wire:
- Make sure the prompt does not block navigation forever; allow "skip" selection or default null if needed (choose and test).
```

---

## Prompt 24 — Skip + lesson + easier drill (end-to-end)

```text
Step 23: Skip + lesson + easier drill.

Goal:
- Add "I'm stuck / Skip" button on Exercise screen.
- On click:
  1) Create an attempt with result="skipped"
  2) Show a short lesson tied to the exercise concept_tags (use exercise.explanation or add a new lesson field)
  3) Navigate to an easier drill exercise (difficulty_variant="easy" for same concept)

Backend:
- Ensure there is at least one easy drill exercise seeded.
- Add endpoint or logic for fetching next recommended exercise variant if needed.

TDD:
- Frontend tests:
  1) Skip button shows lesson view/modal
  2) After lesson, "Try Easier Problem" routes to easy drill
- Backend tests:
  1) Posting skipped attempt works
  2) Content includes easy drill

Wire:
- Entire flow must be usable from the UI.
```

---

## Prompt 25 — Adaptation rules as pure function + integrate into navigation

```text
Step 24: Adaptation rules.

Goal:
- Implement a pure function on frontend (or backend) that decides next activity based on:
  - last mistake_type
  - last confidence (and optionally last 2 confidences)
Rules (MVP):
- If mistake_type is "concept" OR user says "Hard" twice: recommend lesson + easier drill next.
- If mistake_type is "syntax": keep same difficulty, encourage retry.
- If user says "Easy" and passes: reduce scaffolding slightly or advance to next standard exercise.

TDD:
- Unit tests for the pure function with several scenarios.
- Integration test that after a failing run with concept+hard, the app routes to lesson/easy drill.

Wire:
- Make sure the function is actually used to select next screen/exercise.
```

---

## Prompt 26 — Content expansion (Levels 1–2) + polish + demo wiring

```text
Step 25: Content expansion + polish.

Goal:
- Seed 6–10 exercises for Level 1 and Level 2, each with:
  prompt, scaffold_code, tests/harness, 3 hints, explanation, solution_code, concept_tags, and at least one easy drill per concept.
- Ensure quizzes exist for both levels (mixed types).
- Improve UI copy, error states, and navigation:
  - dashboard -> levels -> exercises -> quiz -> unlock next level

Testing:
- Add at least one lightweight end-to-end test (Playwright optional) OR a scripted smoke test in README with commands and expected outputs.
- Ensure unit/integration tests remain stable.

Wire:
- No dead links: every level in GET /levels must be navigable.
- App should be demoable from a fresh signup in under 2 minutes.

Proceed with incremental commits: start by adding Level 1 content then Level 2, validating tests after each.
```

---

## Prompt 27 — Final wiring & hardening pass (no big refactors)

```text
Final wiring & hardening.

Goal:
- Review the codebase for orphaned modules, unused routes, dead UI states.
- Ensure consistency of:
  - attempt.result enums
  - confidence enums
  - progress statuses
- Add rate-limiting / debouncing on Run button to prevent double submissions.
- Add a minimal timeout for code execution (best effort), and a UI message if timeout occurs.

Testing:
- Add/expand tests for:
  - Run button disabled while executing
  - Timeout message path
  - Logout invalidates session
- Ensure all tests pass.

Do not do large refactors. Prefer small fixes and tightening.
End with instructions to run:
- backend tests
- frontend tests
- local dev servers
and a demo script checklist.
```

