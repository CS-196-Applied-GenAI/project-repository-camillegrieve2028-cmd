# Adaptive Coding Coach — Build Blueprint & Iterative Implementation Plan

Owner: Camille Grieve  
Date: 2026-02-25  
Inputs: `spec.md` (project handoff spec)

This document is a **step-by-step blueprint** for building the MVP described in `spec.md`, then a **two-round breakdown** into small, iterative chunks and even smaller implementation steps. The goal is to keep each step:
- **Small enough** to implement safely with strong testing
- **Big enough** to create visible forward progress

---

## 0) Assumptions & Recommended Stack

### 0.1 MVP execution choice
- **Client-side Python execution via Pyodide** (recommended for MVP)
- Backend stores users, content, attempts, progress.

### 0.2 Suggested stack (one practical option)
- **Frontend:** React + TypeScript (Vite)  
- **Backend:** FastAPI (Python) or Node/Express (either is fine)
- **DB:** SQLite for local dev; Postgres if hosted  
- **Auth:** Session cookies or JWT (cookie-based recommended for simplicity)
- **Testing:**
  - Frontend: Vitest + React Testing Library
  - Backend: Pytest (FastAPI) / Jest (Node)
  - E2E smoke: Playwright (optional but great)

> If your course strongly prefers a different stack, keep the plan structure the same and swap the tech.

---

## 1) System Blueprint (How Everything Fits)

### 1.1 Components
1. **UI Web App**
   - Auth pages (signup/login/logout)
   - Dashboard + Level Map
   - Exercise screen (editor + run + hints + skip)
   - Quiz screen (mixed types)
   - Results / progression

2. **Content Service (Backend)**
   - Serves levels, exercises, quizzes
   - Stores attempts and level progress

3. **Execution Engine (Client)**
   - Pyodide loads once
   - “Run” executes user code + tests
   - Captures stdout/stderr/trace
   - Enforces timeout (best effort)
   - Produces verdict (pass/fail) + failure details

4. **Adaptation Engine (Frontend or Backend)**
   - Inputs: mistake type + confidence
   - Outputs: next activity (next exercise / easier drill / lesson)
   - For MVP: implement deterministic rules

### 1.2 Key flows
- **Exercise run:** user code → pyodide execute → verdict + classification → show feedback → persist Attempt → choose next content
- **Hints:** per exercise, hint_count increments → after 3 hints + wrong attempt → show solution/explanation
- **Skip:** triggers lesson → easier drill → persists Attempt status as skipped
- **Quiz:** submit answers → score → update LevelProgress → unlock next level

---

## 2) Content Blueprint (What You Need to Build)

### 2.1 Content authoring format (recommend)
Store content in JSON files checked into repo, then seed to DB (or serve directly as JSON if allowed).

**levels.json**
- id, level_number, title, description

**exercises.json**
- id, level_id, title, prompt, concept_tags
- scaffold_code
- allowed_regions (optional for MVP; can start as simple blanks markers)
- tests (list of assertions) OR expected outputs
- hints (array length >= 3)
- solution_code
- explanation
- difficulty_variant (standard/easy) (optional)

**quiz.json**
- level_id
- questions: type, prompt, options, correct answer, explanation

### 2.2 Minimum content set for MVP demo
- **Level 1:** 6–10 guided exercises + 1 mixed quiz (8–12 questions)
- **Level 2:** 6–10 guided exercises + 1 mixed quiz
- **Optional Level 3:** 4–8 exercises + quiz (stretch)

> It’s better to have 2 polished levels than 3 shallow levels.

---

## 3) Execution & Evaluation Blueprint (Pyodide)

### 3.1 Approach
- For each exercise, provide a **test harness** appended to user code.
- Run in pyodide; capture:
  - stdout
  - exceptions / tracebacks
  - pass/fail based on tests

### 3.2 Example evaluation pattern
- Exercise includes function signature or scaffold.
- Harness imports user definitions and runs asserts:
  - `assert foo(3) == 9`
- If any assert fails → incorrect (logic)
- If SyntaxError/IndentationError → incorrect (syntax)
- If NameError / TypeError may be concept or logic depending on context

### 3.3 Timeout
- Add a soft timeout using:
  - Pyodide interrupt (if available) or
  - run in Web Worker and kill worker on timeout (preferred)
- MVP acceptable: Web Worker-based isolation with a 1–2s timeout.

---

## 4) Data & API Blueprint

### 4.1 Backend tables (from spec)
- Users
- Levels
- Exercises
- QuizQuestions
- Attempts
- LevelProgress

### 4.2 Minimal API endpoints
- POST /auth/signup
- POST /auth/login
- POST /auth/logout
- GET /levels
- GET /levels/{id}/exercises
- GET /levels/{id}/quiz
- POST /attempts
- POST /levels/{id}/quiz-submission
- GET /me (current user + progress)

### 4.3 Validation rules
- Attempts must reference existing user + exercise
- hint_count in [0..3]
- confidence in {easy, ok, hard} or null
- Result in {correct, incorrect, revealed_solution, skipped}

---

## 5) Testing Blueprint (What to Test)

### 5.1 Backend tests (unit/integration)
- Auth:
  - signup creates user
  - login returns session / token
  - protected routes require auth
- Content:
  - levels endpoint returns ordered levels
  - exercises filtered by level
- Attempts:
  - posting attempt stores record
  - invalid payload rejected
- Quiz:
  - scoring correct
  - unlock logic updates LevelProgress

### 5.2 Frontend tests (unit)
- Exercise screen:
  - hint button increments hint count UI
  - after 3 hints + wrong attempt → solution shown
  - skip triggers lesson screen
- Quiz screen:
  - supports MC + guided + free response
  - score computed and posted

### 5.3 Execution tests (unit/smoke)
- A known correct code passes
- A SyntaxError is detected and classified
- A failing assert is detected (logic)
- Timeout path triggers

### 5.4 End-to-end smoke (optional but valuable)
- Signup → Level 1 exercise → complete → take quiz → unlock Level 2

---

# 6) Iterative Build Plan — Round 1 (Chunks)

These are **project chunks** that each produce a meaningful milestone.

## Chunk A — Repo + Dev Environment
Deliverable:
- Monorepo or two repos (frontend/backend)
- Linting, formatting, basic CI test runs

## Chunk B — Backend Skeleton + Database
Deliverable:
- Backend running locally
- DB schema + migrations
- Health check endpoint

## Chunk C — Authentication + “Me” endpoint
Deliverable:
- Signup/login/logout working
- Protected endpoints validated via tests

## Chunk D — Content Delivery (Levels/Exercises/Quiz)
Deliverable:
- Seed content
- GET levels/exercises/quiz endpoints
- Frontend can fetch and render static content list

## Chunk E — Exercise UI (Editor + Run)
Deliverable:
- Exercise screen with scaffolded editor
- Pyodide loads and runs code
- Displays stdout/errors

## Chunk F — Evaluation + Mistake Classification
Deliverable:
- Exercise pass/fail via tests
- Mistake type classification (syntax vs logic vs concept/other)

## Chunk G — Hint System + Solution Reveal
Deliverable:
- 3-hint flow
- Solution/explanation display after limit

## Chunk H — Attempts + Progress Persistence
Deliverable:
- POST attempts
- Track LevelProgress (in_progress, passed)
- Dashboard shows current level

## Chunk I — Quiz UI + Scoring + Unlock Next Level
Deliverable:
- Mixed quiz
- Submit scoring
- Unlock next level when passed

## Chunk J — Adaptation Rules (Within Levels) + Skip Lesson Loop
Deliverable:
- Confidence prompt
- Skip triggers lesson + easier drill selection
- Next exercise selection uses rules

## Chunk K — Polish + Demo Readiness
Deliverable:
- Clean UI
- Seeded content for at least 2 levels
- Full E2E demo path stable

---

# 7) Iterative Build Plan — Round 2 (Break each chunk into smaller steps)

Each chunk is split into steps that can be completed in ~0.5–1.5 days in a student team, with tests.

## Chunk A — Repo + Dev Environment
A1. Initialize repo(s), add README with local run instructions  
A2. Add formatting + lint (prettier/eslint OR ruff/black)  
A3. Add basic test runner + sample test  
A4. Add simple CI workflow (run tests on PR/push)

## Chunk B — Backend Skeleton + Database
B1. Create backend project, /health endpoint  
B2. Add DB connection (SQLite) + config  
B3. Create models/tables (Users, Levels, Exercises, QuizQuestions, Attempts, LevelProgress)  
B4. Add migrations/seed mechanism  
B5. Write backend tests for /health and DB connectivity

## Chunk C — Authentication + “Me”
C1. Implement signup (hash password) + tests  
C2. Implement login (session cookie or JWT) + tests  
C3. Implement logout + tests  
C4. Implement GET /me (requires auth) + tests

## Chunk D — Content Delivery
D1. Define JSON content format and create Level 1 sample content (2 exercises, 2 quiz Qs)  
D2. Seed content into DB (or serve JSON directly)  
D3. Implement GET /levels ordered by level_number + tests  
D4. Implement GET /levels/{id}/exercises + tests  
D5. Implement GET /levels/{id}/quiz + tests  
D6. Frontend: Levels list page renders fetched levels

## Chunk E — Exercise UI + Run
E1. Build Exercise screen layout (prompt, editor, run button, output panel)  
E2. Load Pyodide once (app-level loader) and show loading state  
E3. Implement run execution: execute code, capture stdout/stderr  
E4. Add unit tests for UI (run button disabled while executing; output displayed)

## Chunk F — Evaluation + Classification
F1. For one exercise, append test harness and return pass/fail  
F2. Implement exception parsing for SyntaxError/IndentationError  
F3. Implement assert-failure detection (logic)  
F4. Define mapping rules into mistake_type and display a friendly message  
F5. Add unit tests: correct code passes; syntax fails; assert fails

## Chunk G — Hint System + Reveal
G1. Add “Hint” button + hint_count state per exercise attempt  
G2. Display hint #1/#2/#3 sequentially  
G3. After 3 hints + next incorrect attempt, show explanation + solution  
G4. Add tests for hint progression and solution reveal

## Chunk H — Attempts + Progress
H1. Implement POST /attempts + validation + tests  
H2. Frontend: on every run result, persist Attempt (submitted code + outcome)  
H3. Implement LevelProgress row creation on first interaction + tests  
H4. Dashboard shows current level status (locked/in_progress/passed)

## Chunk I — Quiz + Unlock
I1. Build Quiz screen that supports multiple choice  
I2. Add guided question type (simple fill-in)  
I3. Add free response question type (text area)  
I4. Implement quiz submission endpoint with scoring + tests  
I5. Update LevelProgress to passed when score >= threshold (define threshold)  
I6. Frontend: pass/fail screen and “Go to next level” if passed

## Chunk J — Adaptation + Skip Lesson Loop
J1. Add confidence prompt after exercise (easy/ok/hard) and store with Attempt  
J2. Add “I’m stuck / Skip” flow on Exercise screen  
J3. Create Lesson screen/modal for the relevant concept  
J4. After lesson, route to easier drill exercise  
J5. Implement adaptation rules to choose next exercise based on mistake_type/confidence  
J6. Add tests for adaptation rules (pure functions)

## Chunk K — Polish + Demo
K1. Seed full content for Level 1 & Level 2 (6–10 exercises each, quiz)  
K2. Improve UI copy and error messages  
K3. Add e2e smoke test (optional) or manual demo script checklist  
K4. Final bug bash + performance check (pyodide load, timeouts)

---

# 8) Final Round — “Right-Sized” Micro-Steps (Even Smaller, Safer)

Below are the **smallest safe steps** that still move the project forward, with explicit test expectations. Implement in order.

## Step 1 — Backend boot
1.1 Create backend app + `/health` returns `{status:"ok"}`  
**Test:** health endpoint returns 200 and correct payload.

## Step 2 — User model + migrations
2.1 Add Users table (id, username unique, password_hash, created_at)  
2.2 Migration runs locally  
**Test:** inserting duplicate username fails.

## Step 3 — Signup endpoint
3.1 POST /auth/signup validates payload  
3.2 Hash password and store user  
**Tests:** valid signup works; missing fields rejected; duplicate username rejected.

## Step 4 — Login endpoint
4.1 POST /auth/login verifies password  
4.2 Establish session cookie (or JWT cookie)  
**Tests:** wrong password rejected; logged-in user can access protected route.

## Step 5 — /me endpoint
5.1 GET /me returns current username and basic progress  
**Tests:** 401 if not logged in; 200 with correct user if logged in.

## Step 6 — Levels content minimal
6.1 Add Levels table + seed Level 1  
6.2 GET /levels returns ordered array  
**Tests:** ordering is correct; returns seeded item.

## Step 7 — Exercises content minimal
7.1 Add Exercises table + seed 1 guided exercise  
7.2 GET /levels/{id}/exercises returns seeded exercise  
**Tests:** non-existent level returns empty or 404 (choose one and test).

## Step 8 — Frontend boot + auth screens
8.1 Create React app skeleton + routing  
8.2 Add Signup + Login forms  
8.3 Store session (cookie-based) and call /me on load  
**Tests:** basic component render tests; manual login flow works.

## Step 9 — Level list UI
9.1 Fetch /levels and render list with status placeholders  
**Tests:** handles loading/error states.

## Step 10 — Exercise screen skeleton
10.1 Route: /levels/:id/exercises/:exerciseId  
10.2 Render prompt + editor + run button + output area (no pyodide yet)  
**Tests:** renders exercise content.

## Step 11 — Pyodide loader
11.1 Implement pyodide load service with loading spinner  
11.2 Cache instance globally  
**Tests:** loader returns ready state; UI disables run while loading.

## Step 12 — “Run code” baseline
12.1 Execute code and capture stdout/errors  
12.2 Display output in UI  
**Tests:** a print statement shows output; SyntaxError shows error.

## Step 13 — Exercise evaluation harness
13.1 Append a simple assert test harness for the exercise  
13.2 Produce pass/fail  
**Tests:** known correct submission passes; wrong output fails.

## Step 14 — Mistake classification (MVP)
14.1 Map SyntaxError/IndentationError → syntax  
14.2 Assert failures → logic  
14.3 NameError/TypeError → concept/other (pick mapping and test)  
**Tests:** 3 cases classified correctly.

## Step 15 — Hints (1→3)
15.1 Add hint button and hint display component  
15.2 Ensure hint_count increments and shows correct hint  
**Tests:** clicking hint cycles through hints 1–3.

## Step 16 — Reveal solution after limit
16.1 If hint_count==3 and next attempt incorrect → show explanation + solution  
**Tests:** simulate 3 hints + wrong run triggers reveal.

## Step 17 — Attempt persistence
17.1 Implement Attempts table + POST /attempts  
17.2 Frontend posts attempt on every run  
**Tests:** payload validation; attempt saved; UI handles failure gracefully.

## Step 18 — LevelProgress baseline
18.1 Create LevelProgress when first attempt in level occurs  
18.2 Dashboard shows in_progress for current level  
**Tests:** progress created once; status updates visible.

## Step 19 — Quiz (multiple choice only)
19.1 Seed quiz questions for Level 1 (MC only)  
19.2 Quiz screen renders MC + submit  
19.3 Backend scores MC and returns score  
**Tests:** correct scoring; fail when missing answers.

## Step 20 — Quiz unlock
20.1 Define pass threshold (e.g., 70%)  
20.2 If pass, update LevelProgress to passed and unlock Level 2  
**Tests:** pass updates; fail does not.

## Step 21 — Add guided + free response quiz items
21.1 Implement guided question scoring (exact match or simple normalization)  
21.2 Implement free response scoring (MVP: rubric keywords OR manual review flag)  
**Tests:** normalization works; scoring deterministic.

## Step 22 — Confidence capture
22.1 Add “Easy/OK/Hard” prompt after exercise completion  
22.2 Store confidence on Attempt  
**Tests:** attempt includes confidence; UI requires selection or allows skip (choose and test).

## Step 23 — Skip + lesson + easier drill
23.1 Add Skip button  
23.2 Show lesson content tied to concept tag  
23.3 Route to easier drill exercise  
**Tests:** skip creates attempt with status skipped; lesson appears; next exercise loaded.

## Step 24 — Adaptation rules (pure function)
24.1 Implement `chooseNextActivity({mistake_type, confidence_history})`  
24.2 Integrate into exercise navigation  
**Tests:** pure unit tests for rule cases; integration test for one rule.

## Step 25 — Content expansion + polish
25.1 Add 6–10 exercises for Level 1 and Level 2  
25.2 Improve empty/error states  
25.3 Add demo checklist script  
**Tests:** smoke test flows pass.

---

## 9) Definition of Done (MVP Demo Checklist)
- User can sign up and log in
- Can complete guided Python exercises with real execution
- Gets hints (up to 3), then explanation + solution
- Can skip and receive a short lesson + easier drill
- Can take a mixed quiz and unlock next level
- Progress persists across refresh/login

---

## 10) Notes on “Right-Sizing”
If any step feels too big, split it further by:
- Implementing one endpoint without persistence (mock), then adding DB save
- Implementing one question type at a time
- Shipping one level with 2–3 exercises, then expanding content

