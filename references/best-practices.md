# Best Practices (Phase 3 & 4)

## Locator strategy (priority order)

1. `get_by_role(...)` — most resilient, mirrors how users/assistive tech find elements.
2. `get_by_label(...)`, `get_by_placeholder(...)`, `get_by_text(...)` — semantic, readable.
3. `get_by_test_id(...)` — if the app exposes `data-testid` attributes, use them for elements with no good semantic role/text.
4. CSS/XPath selectors — last resort only, and only when nothing above is feasible (e.g. third-party widget markup you can't change).

Never use brittle selectors like nth-child chains or auto-generated class names (`.css-1x2y3z`) if any alternative exists.

## Waiting strategy

- Never use bare `time.sleep()` / fixed waits. Playwright auto-waits for actionability on actions like `.click()`/`.fill()` — lean on that.
- For navigation, use `page.wait_for_url(...)` or `expect(locator).to_be_visible()` rather than guessing a delay.
- For network-dependent state, prefer waiting on a UI signal (element appears/disappears) over `wait_for_load_state("networkidle")` alone, which can be unreliable on apps with polling/websockets.

## Test independence & data

- Each test must be runnable alone and in any order — no test should depend on state left behind by another. Use fixtures to set up needed state per test (or per class via fixture scope), not by relying on execution order.
- Generate unique test data per run where uniqueness matters (e.g. emails) — use a factory/helper (`fixtures/test_data.py`), e.g. timestamp or uuid suffixes, rather than hardcoded literals that collide on reruns.
- Clean up created data where feasible (teardown in fixture), especially for anything that pollutes shared environments.

## Structure & naming

- One assertion focus per test — a test should answer one question ("does valid login work?") even if it contains multiple `assert` lines verifying that one outcome.
- Arrange-Act-Assert layout, in that order, inside every test.
- Test names describe behavior: `test_<scenario>_<expected_outcome>`, e.g. `test_empty_cart_checkout_shows_warning`.
- Use `@pytest.mark.parametrize` for variants of the same flow (valid/invalid inputs, different roles) instead of copy-pasted near-duplicate tests.
- Mark slow/critical tests (`@pytest.mark.smoke`, `@pytest.mark.slow`) so the user can run subsets.

## Scalability — adding a new flow later

Document this in the wrap-up so it's obvious: adding a flow means (1) add/extend a Page Object in `pages/`, (2) add a test file in the matching `tests/<feature>/` folder, (3) add any new fixtures/data builders if needed. Nothing in the existing suite should require editing.

## Pre-handoff code-quality checklist (run through this before Phase 5)

- [ ] No locator string is duplicated across more than one file — it lives in exactly one Page Object.
- [ ] No `sleep()`/fixed waits anywhere in the suite.
- [ ] No hardcoded secrets, URLs, or environment-specific values — all via `.env`/config.
- [ ] Every test file has a clear, single feature focus; no god-files mixing unrelated flows.
- [ ] Suite passes twice in a row from a clean state (no flakiness).
- [ ] `.env.example` documents every required env var, with no real values filled in.
- [ ] README or wrap-up message explains how to run tests and add new ones.