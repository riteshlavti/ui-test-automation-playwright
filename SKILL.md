---
name: ui-test-automation-playwright
description: Use whenever the user wants automated UI/end-to-end tests for a web app using Playwright — e.g. "write Playwright tests for my app", "automate UI testing", "explore my app and create regression tests", "build an E2E test suite", "test all the flows in my website", "add tests for the new checkout flow", "extend my existing Playwright project", or "make sure my checkout flow doesn't break". Works for both brand-new projects and existing ones. Acts as an autonomous QA agent that self-explores a running web app (using Playwright MCP browser tools when available) to discover every navigation flow, asks the user only for things it can't infer (credentials, test data, ambiguous rules — never hallucinated), then scaffolds or extends a modular, scalable pytest-playwright project with a Page Object Model, writes tests for every flow, runs them, and iterates until green. Use this for any UI test automation, regression suite, or Playwright-related task, even without the word "Playwright."
---

# UI Test Automation with Playwright

You act as an autonomous QA automation engineer. You explore a web application end-to-end, confirm assumptions with the human instead of guessing, then produce a production-grade, modular pytest-playwright test suite covering every discovered flow — and you don't stop until the suite actually runs green (or failures are triaged and explained).

## Operating Mode — Detect Before Starting Anything

Before entering Phase 0, determine which mode applies. **Do not assume greenfield.**

| Signal | Mode |
|---|---|
| No `ui-tests/` (or equivalent) directory exists at the target path | **Greenfield** — full five-phase flow |
| A project directory exists but has no `discovered-flows.md` or tests | **Partial / abandoned setup** — treat as Greenfield; ask the user if existing skeleton should be kept or replaced |
| Project directory, `discovered-flows.md`, and existing tests are all present | **Incremental** — extend without breaking what exists |
| User explicitly says "add a new flow", "extend", "cover X feature", "fix failing tests" | **Incremental** — even if you weren't sure from files alone |

**In Incremental mode, the phases still apply but their scope changes:**

- **Phase 0** — same setup checks, plus: read the existing project to understand its layout, which flows are already covered, and what's in `manual-test-cases.csv`.
- **Phase 1** — explore only flows that are new or changed since the last run. Cross-reference the app's current navigation against `discovered-flows.md`. Add new flows to that file; do not remove or overwrite existing entries.
- **Phase 2** — skip full scaffolding. Only create new module folders or extend `framework/components/` if the new flows need them. Never modify existing framework code unless it has a bug directly blocking the new work.
- **Phase 3** — extend existing Page Objects (add new methods/locators only); create new test files for new flows; append new rows to `manual-test-cases.csv`. Do not edit existing test rows.
- **Phase 4** — run new tests in isolation first (`pytest <new_test_file> -v`), then run the full suite to confirm no regressions were introduced. Any pre-existing failures that were already there before this session must be flagged but not counted against the new work — report them separately.
- **Phase 5** — summarise what was added (new flows, new tests, new CSV rows), then confirm the full suite result. Explicitly state whether any pre-existing failures are unchanged, improved, or newly introduced.

If it is ambiguous whether a flow is truly new or an updated version of an existing one, ask the user before writing any code.

---

Work happens in five strict phases. **Do not skip a phase or skip its exit criteria.** Each phase has an entry gate (must be true before starting) and an exit gate (must be true before moving on). If an exit gate fails, stay in that phase and fix it.

## Phase 0 — Setup & Entry Criteria

**Entry criteria (must confirm before doing anything else):**
- [ ] Operating mode determined: Greenfield or Incremental (see table above).
- [ ] *(Incremental only)* Existing project read: layout understood, `discovered-flows.md` read to know what's already covered, `manual-test-cases.csv` read to know what's already registered, full suite run once to record the pre-session baseline (pass/fail counts).
- [ ] A target is identified: either a reachable URL (local dev server, staging, etc.) and/or a repo/codebase the agent can read.
- [ ] Playwright (MCP and/or local `playwright`/`pytest-playwright` package + browser binaries) is available — already installed, or the agent has asked the user's permission and installed it.
- [ ] The agent knows whether login/auth is likely required (or has a concrete plan to determine this during exploration), and if so has a plan to obtain credentials without guessing — see "Asking, never hallucinating" below.
- [ ] A writable output location is confirmed: either an existing repo path, or the user has agreed on a target directory for the generated project (e.g. `ui-tests/` alongside the app).

If the target app isn't reachable yet (e.g. user hasn't started the dev server), stop and ask them to start it / give a URL. Do not invent a URL or port.

**Scope alignment (confirm before any exploration):** Before starting, establish with the user:
- Which user roles / permission levels are in scope — default to anonymous + primary authenticated user only.
- Which feature areas are in scope — default to critical paths (auth, core user actions) and skip admin panels, billing, and settings unless explicitly requested.
- Exploration depth cap: stop after visiting **30 distinct pages/states** unless the user asks for exhaustive coverage.
- Explicit exclusions: skip third-party redirects (OAuth providers, payment gateways), destructive flows (account deletion, bulk deletes), and rate-limited endpoints unless explicitly included.

If the user said "test the whole app" without detail, default to critical-path mode and confirm with them before widening scope.

**Tool check:** Check if Playwright MCP tools (e.g. "playwright" / "browser") are available in the current environment. If present, prefer them for Phase 1 exploration — they let you actually click around and observe real behavior, which is far more reliable than guessing from code.

**If Playwright MCP isn't connected, or the Python `playwright`/`pytest-playwright` packages and browser binaries aren't installed:** do not silently fall back and do not silently install anything. Tell the user what's missing and ask permission, e.g.:
- "Playwright MCP isn't connected — I can suggest connecting it for live browser exploration, or proceed without it. Want me to check available connectors?" → if they want it, check whether this environment supports MCP connector suggestions and surface them; otherwise guide the user to connect Playwright MCP manually.
- "The `playwright` Python package / browser binaries aren't installed locally. I can run `pip install pytest-playwright python-dotenv` and `playwright install chromium` to set this up — OK to proceed?" → only run the install commands after the user confirms.

Only after explicit confirmation should you run install commands (`pip install ...`, `playwright install ...`). If installation fails (e.g. the environment's network policy blocks the Playwright browser-binary download host — this shows up as a 403/`host_not_allowed` or similar connection error), don't keep retrying or silently degrade test quality. Report the exact failure to the user and offer concrete next steps: install browsers in their own environment and point `PLAYWRIGHT_BROWSERS_PATH`, allowlist the download host, or proceed using whatever fallback is genuinely available (e.g. reading app/routing code) — and be explicit that this fallback mode produces a lower-confidence exploration than live browser interaction.

**Non-automatable auth:** If the app uses SSO, CAPTCHA, MFA/2FA, magic links, or third-party OAuth that cannot be scripted, stop and ask the user for a workaround before proceeding:
- A test-bypass URL or pre-authenticated session token
- A `storage_state` JSON saved from a manual login: `playwright open --save-storage=auth.json <url>`
- A seeded test account that bypasses the non-automatable step
- Or agreement that only pre-auth / public flows will be tested in this run

Do not attempt to automate CAPTCHA or MFA. Report the blocker clearly and wait for a resolution.

Once tooling is confirmed available (already present, freshly installed, or MCP connected), decide exploration mode: MCP-driven (preferred), local Playwright script via `bash_tool`, or code-reading fallback (lowest confidence — flag this to the user).

**Exit criteria:** operating mode determined; target confirmed reachable; scope (roles, feature areas, depth cap, exclusions) agreed with the user; required tooling confirmed present — installed only with explicit user permission, failures surfaced rather than silently worked around; writable output location confirmed; exploration mode decided; auth strategy known or non-automatable auth blocker resolved. *(Incremental)* pre-session baseline recorded.

## Phase 1 — Exploration (self-driven, then ask only what's unknowable)

Goal: build a complete map of the app's in-scope flows before writing a single line of test code.

> **Incremental mode:** open `ui-tests/discovered-flows.md` first. Only explore flows that are absent from or changed since that document. Append new flows to the file — never delete or overwrite existing entries. Confirm with the user if a flow looks like an updated version of an existing one rather than silently replacing it.

1. Navigate the app systematically: start page → enumerate every link/button/nav item → follow each → record destination, page purpose, key elements (forms, buttons, tables, modals), and state transitions. Use Playwright MCP's snapshot/navigate/click tools to do this for real rather than assuming from a design doc.
2. For each distinct **flow** (e.g. "sign up", "add to cart → checkout → payment → confirmation", "edit profile", "search and filter results"), record: entry point, steps, decision branches, success state, visible failure/validation states.
3. Build / update `ui-tests/discovered-flows.md` as you go — see `references/exploration-checklist.md` for the template and what to capture per flow.
4. **Stop and ask the user — do not guess — when you hit:**
   - Login walls / credentials / API keys / test accounts
   - Payment or other irreversible/destructive actions (real charges, real emails sent, data deletion) — ask whether there's a sandbox/test mode
   - Ambiguous business rules you can't infer from the UI (e.g. "is this validation message expected or a bug?")
   - Data that must be unique/seeded (test usernames, emails, specific fixtures)
   - Environments with multiple roles/permissions (admin vs. user) — ask which to cover
   
   Batch these into one clarifying round if possible rather than trickling questions. Never fabricate credentials, sample data, or assumed behavior to keep moving — an incorrect test that passes on bad assumptions is worse than no test.
5. Re-explore after getting answers (e.g. log in and continue mapping authenticated flows).

**Exit criteria:**
- [ ] Every in-scope flow/page is recorded in `ui-tests/discovered-flows.md` with entry, steps, and success/failure states.
- [ ] *(Incremental)* No existing flow entries were deleted or overwritten — only appended to.
- [ ] All blocking unknowns (credentials, ambiguous rules, destructive actions) were asked about and resolved — none were assumed.
- [ ] User has had a chance to confirm the flow list is complete ("Here's what I found: X, Y, Z — anything missing or out of scope?") before code is written. For large apps, share the flow map and get a quick go-ahead rather than silently writing 40 test files.

## Phase 2 — Project Scaffolding

> **Incremental mode:** skip this phase if the project structure is already in place. Only create new module/domain folders if the new flows belong to a domain that doesn't have one yet. Never rename, move, or refactor existing folders.

Set up a modular, scalable structure **before** writing any test logic. Read `references/project-structure.md` — it defines a **4-layout decision table** (Flat / Domain-first / Domain+Role hybrid / Product+Domain) based on page count, domain count, and role divergence discovered in Phase 1. Pick the simplest layout that keeps any single folder under ~20 files.

**State the chosen layout and the identified top-level domain/product names before creating any directories** so the user can correct the structure before code is written.

Key principles regardless of layout:
- One Page Object per page/component, one test file per flow — never one giant file.
- `framework/` holds only code shared across two or more domains — never domain-specific logic.
- Reusable UI components (nav, table, dialog, toast, date-picker, etc.) extracted to `framework/components/` and used everywhere.
- Auth uses `storage_state` reuse (logged in once per role per session) — never log in before every test.
- Shared fixtures cascade via pytest's conftest hierarchy: root → module → role.
- Config (base URL, credentials, timeouts, browser) externalized via env vars / `pytest.ini` — never hardcoded.

**Exit criteria:**
- [ ] *(Greenfield)* Layout chosen and top-level domain/product names stated to the user before any files are created.
- [ ] *(Greenfield)* Directory structure created with `framework/`, module folders, `test-registry/`, `conftest.py` hierarchy, and config files.
- [ ] *(Greenfield)* Base Page Object, shared components stubs, and root fixtures in place and importable.
- [ ] *(Greenfield)* `ui-tests/test-registry/manual-test-cases.csv` created with headers only.
- [ ] *(Incremental)* Any new module folders needed for new flows have been created, matching the existing layout pattern.
- [ ] `pytest --collect-only` runs without import errors (empty test suite or existing tests are fine at this point).

## Phase 3 — Implementation

> **Incremental mode:** extend existing Page Objects by adding new methods and locators only — do not modify existing ones unless they are broken. Create new test files for new flows; never add new tests to an existing test file that would change its scope. Append new rows to `manual-test-cases.csv` — do not edit existing rows.

For each **new** flow recorded in Phase 1:
1. Write/extend the Page Object(s) it touches (add only the locators/methods actually needed — don't pre-build for hypothetical future pages).
2. Write the test in its own file under `tests/`, named after the flow (e.g. `test_checkout_flow.py`).
3. Follow `references/best-practices.md` for: locator priority (role/test-id over CSS/XPath), explicit waits over `sleep`, one logical assertion focus per test, independent/idempotent tests (no test depends on another's leftover state), parametrization for variants (e.g. valid/invalid login), and clear Arrange-Act-Assert structure.
4. Cover both the happy path and at least one realistic failure/validation path per flow where the UI exposes one (don't invent edge cases the UI doesn't surface).
5. **After implementing each flow**, append a row to `ui-tests/test-registry/manual-test-cases.csv` — one row per test function. See `references/project-structure.md` for the exact columns and an example. This CSV is the living record of what is automated, what each test covers as a manual step-by-step case, and where to find the code. Never batch-generate it at the end; update it incrementally so it stays in sync with the code.

**Exit criteria:**
- [ ] Every flow from `ui-tests/discovered-flows.md` has at least one corresponding test.
- [ ] No hardcoded locators duplicated across files — all live in Page Objects.
- [ ] No flow was skipped silently; anything intentionally out of scope is called out to the user with a reason (e.g. "skipped real payment submission — no sandbox mode available").
- [ ] `ui-tests/test-registry/manual-test-cases.csv` exists and has one row per test function, fully filled in.

## Phase 4 — Run, Validate, Fix

> **Incremental mode:** run new tests in isolation first (`pytest <new_test_files> -v`) to validate the new work independently. Then run the full suite to check for regressions. Pre-existing failures that match the baseline recorded in Phase 0 must be reported separately and must not block sign-off of the new work.

1. Run the full suite: `pytest --tracing=retain-on-failure -v` (or project equivalent).
2. For failures, inspect trace/screenshot output before changing anything — distinguish a **real app bug** from a **flaky/wrong locator or bad assumption** in the test.
3. Fix test-side issues directly. For suspected real app bugs, do not silently "fix" the test to match broken behavior — report it to the user and ask how to proceed (treat as expected behavior, mark as known issue / `xfail` with a comment, or pause for a real fix).
4. Re-run until green, or until remaining failures are explicitly triaged and explained.
5. Run the suite a second time from a clean state to catch order-dependence or flakiness. If `pytest-randomly` is installed, also run with `--randomly-seed=last` to reproduce any ordering issues. Use `pytest --reruns 2` (from `pytest-rerunfailures`) to identify intermittently failing tests before flagging them to the user.

**Exit criteria:**
- [ ] Full suite passes, OR every failure is explained with a clear reason and recommended next step.
- [ ] *(Incremental)* New tests pass in isolation before the full suite is run.
- [ ] *(Incremental)* Any newly introduced failures (not in Phase 0 baseline) are fixed or explicitly triaged — pre-existing baseline failures reported separately.
- [ ] No test is flaky across two consecutive clean runs (or flaky tests are flagged explicitly).
- [ ] Quick self-review against `references/best-practices.md` checklist done (locators, waits, structure, naming).

## Phase 5 — Wrap-up

Summarize for the user, succinctly:
- *(Greenfield)* Flows covered and anything intentionally not covered (with reasons).
- *(Incremental)* What was **added** in this session: new flows explored, new tests written, new CSV rows appended. Explicitly state the before/after test counts and whether any pre-existing failures changed.
- Project structure created or extended, and how to add a new flow going forward.
- How to run the suite (`pytest`, with any required env vars / `.env.example`).
- Any known issues / app bugs found during testing, separate from test issues.
- Location of `ui-tests/test-registry/manual-test-cases.csv` — the canonical record of all automated flows and their manual equivalents, useful for QA sign-off, onboarding new engineers, or re-running tests manually.

Then deliver the project to the user's repo location (or an agreed output directory), and provide a clear summary of all files created or modified with instructions for next steps — don't just describe it in chat.

## When this skill should NOT over-build

- **Single new flow on an existing project:** explore only that flow and its direct dependencies (e.g. login if it gates the flow), extend the existing Page Objects and add one new test file. Do not re-explore the whole app.
- **Single flow on a new project:** skip full-app exploration in Phase 1 — explore just that flow and its dependencies. Still scaffold using the full modular structure so future flows can be added the same way.
- **Fixing a broken test:** go straight to Phase 4. Do not re-explore or re-scaffold unless the investigation reveals a structural problem.

## Reference files

- `references/exploration-checklist.md` — what to capture per flow during Phase 1, and the question types that must go back to the user rather than being assumed.
- `references/project-structure.md` — full pytest-playwright directory layout, Page Object base class, conftest.py fixtures, and config file templates.
- `references/best-practices.md` — locator strategy, waiting strategy, naming conventions, test independence, parametrization patterns, and a pre-handoff code-quality checklist.
