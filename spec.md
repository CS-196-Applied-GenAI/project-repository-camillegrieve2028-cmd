# Adaptive Coding Coach — MVP Specification (Python, Beginner-Focused)

Owner: Camille Grieve  
Date: 2026-02-25  
Document: Developer handoff spec for a class project MVP

---

## 1. Product Overview

### 1.1 One-liner
An adaptive, beginner-friendly Python coding practice app that uses guided exercises, targeted hints, and short concept lessons to help intro CS learners and self-taught beginners build core coding logic skills.

### 1.2 Target users
- **Primary:** Intro CS students (early coursework)
- **Also:** Career switchers / self-taught beginners

### 1.3 Primary learning goals (all are in-scope)
- Teach core programming concepts (especially logic)
- Improve problem-solving thinking
- Build confidence and reduce frustration
- Provide personalized progression (within a simple level structure)

---

## 2. MVP Scope Decisions (Locked)

### 2.1 Language
- **Python only** for MVP.

### 2.2 Primary learning mode
- **Guided / fill-in-the-blank coding** is the main interaction model.

### 2.3 Adaptation signals
- **Primary driver:** **Mistake type** (concept vs syntax vs logic vs off-by-one, etc.)
- **Secondary input:** **User confidence** / self-reported difficulty (“easy / hard”)

### 2.4 Wrong-answer behavior
- On incorrect attempt: **show a hint and let user try again**.
- **Hint limit:** 3 hints max, then provide **full explanation + solution**.

### 2.5 Curriculum starting point
- Start with **core logic**: conditionals, boolean logic, loops, comparisons.

### 2.6 Progress structure
- **Linear levels**: Level 1 → Level 2 → Level 3 …

### 2.7 Level unlock mechanism
- Unlock next level by **passing a Level Quiz**.

### 2.8 Quiz format
- **Mixed quiz**:
  - Some multiple-choice questions
  - Some guided fill-in-the-blank
  - Some free-response (less scaffolded)

### 2.9 Accounts / persistence
- **Username + login** required.
- Progress is saved per user.

### 2.10 Correct-answer feedback style
- After correct answers, provide **concept reinforcement + a quick learning tip**.

### 2.11 Long-term “mistake memory”
- **Out of scope for MVP.** (No cross-session mistake analytics or pattern memory.)

### 2.12 Skipping behavior
- **Adaptive skip**:
  - If user is stuck, allow skip **with** a short concept lesson.
- After skip: **short targeted lesson** + then a **simpler practice problem**.

### 2.13 Real code execution
- **Yes:** execute Python code for evaluation (not just pattern matching).

---

## 3. Open Question (Not Resolved)

### 3.1 Where Python code executes
This was asked but not answered before ending questions. To keep the MVP buildable for a class project, the recommended default is:

- **Default assumption for MVP:** Run Python **in the browser** using Pyodide (client-side execution).
  - Rationale: avoids server-side sandbox/security complexity while still providing “real execution”.

**Alternative (upgrade path):** server-side sandbox execution via backend containers/firejail + resource limits.

> If your team already has strong backend infra experience, you may choose server-side execution instead; otherwise use Pyodide.

---

## 4. User Experience (UX) Requirements

### 4.1 Core user journeys

#### A) New user onboarding
1. User lands on welcome screen
2. Creates account (username + password) or logs in
3. Selects “Start Level 1”
4. Sees first guided exercise

#### B) Practice loop (guided exercise)
1. User sees problem prompt + partially filled code scaffold
2. User fills blanks / edits within allowed regions
3. User runs code (“Run” button)
4. System evaluates output/behavior
5. If correct:
   - Show “Correct” + quick concept tip
   - Advance to next exercise
6. If incorrect:
   - Show hint (attempt 1/2/3)
   - Allow retry
7. If incorrect after 3 hints:
   - Show explanation + solution
   - Mark as “completed-with-solution”
   - Continue forward (no blocking)

#### C) Skip flow (stuck)
- If user triggers “I’m stuck / Skip” (or system detects repeated failures):
  1. Provide a short targeted lesson (1–2 minutes read time)
  2. Provide an easier exercise that practices the same concept

#### D) Level quiz flow
- User initiates quiz at end of level
- Mixed question types
- Pass/fail result
- If fail:
  - Recommend 2–3 targeted exercises to retry within the level
  - Allow retake after completing those

---

## 5. Functional Requirements

### 5.1 Authentication
- Signup: username, password
- Login: username, password
- Logout
- Persist session (token/cookie)

### 5.2 Levels & content
- Levels are a fixed ordered sequence.
- Each level has:
  - A set of guided exercises
  - A quiz

### 5.3 Guided exercise engine
Each exercise includes:
- Title
- Prompt (plain text + optional examples)
- Concept tags (e.g., conditionals, loops, boolean logic)
- Code scaffold template
- Allowed edit regions (blanks or editable lines)
- Unit tests / evaluation function
- Hint set (at least 3 hints)
- Explanation & solution

### 5.4 Hint system
- Track hint count per exercise attempt.
- Show next hint on request (“Hint” button) OR automatically after wrong attempt (choose one implementation; recommended: user clicks hint).
- After hint #3 and another wrong attempt, reveal explanation + solution.

### 5.5 Confidence input
- After each exercise (or after each level), prompt:
  - “How did that feel?” (Easy / Okay / Hard)
- Store this per exercise attempt for adaptation.

### 5.6 Adaptation logic (MVP)
Even though levels are linear, within a level the next exercise can adapt.

**Inputs**
- Mistake type classification (see below)
- Confidence rating

**Outputs**
- Select next exercise variant:
  - Standard difficulty
  - Easier variant (more scaffolding)
  - Extra practice drill
  - Mini lesson (triggered on skip or repeated confusion)

**Minimum adaptation rules**
- If mistake is “concept misunderstanding” OR user says “Hard” twice:
  - Serve a mini lesson + an easier drill next
- If mistake is “syntax small error”:
  - Serve a hint focused on syntax and allow retry; keep same difficulty
- If user consistently says “Easy” and gets correct:
  - Reduce scaffolding slightly in subsequent exercises (within the same level)

### 5.7 Mistake type classification (MVP heuristic)
Because full analytics is out of scope, implement a simple classifier:
- SyntaxError / indentation error → **syntax**
- Wrong output but runs → **logic**
- Off-by-one patterns (common in loops) → **logic**
- Using wrong operator / boolean condition → **concept**
- Timeout / infinite loop (if detectable) → **logic**

If using Pyodide, capture:
- stdout
- stderr
- exception trace
- runtime limit exceeded (if implemented)

---

## 6. Non-Functional Requirements

### 6.1 Performance
- Code execution should return results within ~1–3 seconds for typical beginner problems.

### 6.2 Safety & security (if client-side)
- Client-side execution reduces server risk, but still:
  - Prevent access to browser APIs beyond the editor (standard sandbox behavior)
  - Limit runtime to avoid infinite loops (timeout)

### 6.3 Accessibility
- Keyboard navigable editor
- Clear error messages
- High-contrast friendly UI

---

## 7. Data Model (Minimum)

### 7.1 Entities

**User**
- id
- username
- password_hash
- created_at

**Level**
- id
- level_number
- title
- description

**Exercise**
- id
- level_id
- title
- prompt
- scaffold_code
- evaluation_type (tests / expected_output)
- hints (array)
- solution_code
- explanation

**QuizQuestion**
- id
- level_id
- type (multiple_choice | guided | free_response)
- prompt
- options (if MC)
- correct_answer / rubric
- explanation

**Attempt**
- id
- user_id
- exercise_id
- timestamp
- submitted_code
- result (correct | incorrect | revealed_solution | skipped)
- hint_count (0–3)
- mistake_type (syntax | logic | concept | other)
- confidence (easy | ok | hard)

**LevelProgress**
- id
- user_id
- level_id
- status (locked | in_progress | quiz_unlocked | passed)
- quiz_score
- updated_at

---

## 8. APIs / Interfaces

### 8.1 If using client-side execution (recommended)
- No “run code” backend endpoint required.
- Backend provides:
  - Auth endpoints
  - Content endpoints (levels/exercises/quizzes)
  - Save attempt/progress endpoints

**Example endpoints**
- POST /auth/signup
- POST /auth/login
- GET /levels
- GET /levels/{levelId}/exercises
- GET /levels/{levelId}/quiz
- POST /attempts (save attempt metadata + code + outcome)
- POST /levels/{levelId}/quiz-submission

### 8.2 If using server-side execution (alternative)
- POST /execute
  - input: code + test spec
  - output: pass/fail, stdout, stderr, trace, runtime

---

## 9. UI Screens (MVP)

1. **Landing / Auth**
   - Signup, Login
2. **Home / Dashboard**
   - Current level, progress indicator
   - “Continue”
3. **Level Map**
   - Level list (locked/unlocked/passed)
4. **Exercise Screen**
   - Prompt, scaffolded editor, Run, Hint, Skip
   - Output panel + feedback panel
5. **Lesson Modal / Lesson Page**
   - Short concept lesson + “Try Easier Problem”
6. **Quiz Screen**
   - Mixed question types
7. **Results**
   - Pass/fail, what to do next

---

## 10. Content Requirements (Starter Set)

### 10.1 Level 1: Conditionals + boolean logic
- Exercises: comparisons, if/else, elif, boolean operators
- Quiz: MC on boolean expressions + guided + 1 free response

### 10.2 Level 2: Loops basics
- Exercises: for loops, while loops, counting, accumulation
- Quiz: mixed

### 10.3 Level 3: Loops + conditionals together
- Exercises: filtering, simple patterns
- Quiz: mixed

> Note: Functions can be Level 4+ (post-MVP or stretch).

---

## 11. Acceptance Criteria

### 11.1 Core
- Users can sign up, log in, and have persistent progress.
- Users can complete guided exercises with real Python execution.
- Wrong answers provide hints; after 3 hints, solution/explanation is shown.
- Users can take a quiz; passing unlocks the next level.

### 11.2 Adaptation
- System records mistake type + confidence.
- System uses at least the “minimum adaptation rules” to choose next step (easier drill / lesson).

---

## 12. Future Enhancements (Out of Scope for MVP)
- Long-term “mistake memory” and personalized analytics dashboard
- More languages
- Interview prep mode
- Spaced repetition scheduling
- Social features (friends, leaderboards)

---

## 13. Assumptions & Notes
- Because server-side sandboxing is complex for a class project, **Pyodide (client execution)** is assumed unless your team explicitly chooses server execution.
- Content can be stored as JSON seeded into a database or served from static files for MVP.

