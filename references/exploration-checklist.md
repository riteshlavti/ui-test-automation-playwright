# Exploration Checklist (Phase 1)

## Per-flow template — fill this in for every discovered flow

```
### Flow: <short name, e.g. "User Signup">
- Entry point: <URL / button / nav item that starts it>
- Preconditions: <e.g. must be logged out; cart must have items>
- Steps:
  1. ...
  2. ...
- Decision branches: <e.g. "if email already exists -> error state X">
- Success state: <what confirms success — redirect, toast, DB change visible in UI>
- Visible failure/validation states: <field errors, disabled buttons, etc.>
- Data needed: <fields, formats, whether values must be unique>
- Role/permissions required: <anonymous / user / admin / etc.>
- Notes/risks: <destructive? rate-limited? third-party redirect (OAuth, payment)?>
```

Keep one of these per flow in `discovered-flows.md`. This file is the source of truth Phase 3 implements against — don't start writing tests until it's reasonably complete and the user has glanced at it.

## How to explore systematically with Playwright MCP

1. Navigate to the base URL. Take a snapshot/screenshot to see the landing state.
2. Enumerate all interactive elements on the page (nav links, buttons, menu items).
3. Maintain a queue of "discovered but unvisited" destinations and a set of "visited" URLs/states to avoid infinite loops (watch for pagination, infinite scroll, or "back to same page" links).
4. For forms, identify required fields by inspecting `required` attributes / submitting empty and reading validation — don't submit real-looking data until you know what's safe (see destructive-action rule below).
5. For multi-step flows (wizards, checkout), walk the entire sequence once to record it, without necessarily completing irreversible final steps (e.g. stop before final "Place Order" if it would charge a real card, unless told there's a sandbox).
6. Note any flows gated by auth/roles — map what's visible per role if multiple roles exist.

## Questions that MUST go back to the user (never assume)

- Login credentials or test accounts (even "is there a seeded test user?")
- API keys / third-party service credentials (payment gateways, OAuth providers)
- Whether a payment/checkout flow has a sandbox/test mode, or should be tested only up to the point before submission
- Specific data formats required by the backend that aren't obvious from the UI (e.g. phone number format, required ID patterns)
- Whether destructive actions (delete account, delete data) are safe to exercise against this environment
- Which user roles/permission levels are in scope
- Anything where two reasonable interpretations of expected behavior exist

If you find yourself about to type a plausible-looking password, email, or API key into a test "just to get something working" — stop. That is the signal to ask instead.