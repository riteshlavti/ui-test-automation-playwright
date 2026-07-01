# Project Structure (Phase 2)

## Goal

Create a production-ready, modular pytest-playwright project that scales cleanly from a handful of pages to hundreds. The structure **must be generated from what was discovered in Phase 1** — not from a fixed template. The agent decides the layout; this document defines the rules for making that decision correctly.

---

## Step 1 — Choose the Segregation Strategy

Before creating any files, classify the application from the Phase 1 flow map, then pick the matching strategy:

| Signal from Phase 1 | Strategy | When to use |
|---|---|---|
| ≤15 pages, 1–2 roles, 1 domain | **Flat** | Small or focused apps — e.g. a public checkout funnel, a single-module admin tool |
| 15–60 pages, clear business domains (Orders, Users, Reports…), ≤3 roles | **Domain-first** | Most enterprise SaaS products — organise by business capability |
| 60+ pages or multiple distinct user roles where the same domain looks very different per role | **Domain + Role hybrid** | Large enterprise portals, multi-tenant platforms — domain is the top level, role is a sub-level within domains that diverge significantly |
| App is a multi-product suite or has independently deployable sub-apps | **Product + Domain** | Conglomerates, super-apps — each product is a top-level module, domains live inside |

**Decision rule:** choose the simplest strategy that keeps any single folder from growing beyond ~20 files. If in doubt, use Domain-first and refactor into hybrid only if a domain folder exceeds that threshold.

State the chosen strategy and the identified top-level domain/role names at the start of Phase 2, before creating any directories, so the user can correct it if needed.

---

## Step 2 — Apply the Matching Layout

### Layout A — Flat (small apps)

```
ui-tests/
├── pytest.ini
├── conftest.py
├── .env.example
├── requirements.txt
├── discovered-flows.md
├── framework/
│   ├── __init__.py
│   ├── base_page.py
│   ├── components/          # reusable UI components (nav, table, dialog, etc.)
│   │   └── __init__.py
│   └── utils/
│       └── test_data.py
├── pages/                   # all page objects in one layer
│   ├── __init__.py
│   ├── login_page.py
│   └── dashboard_page.py
├── tests/                   # all test files in one layer
│   ├── __init__.py
│   ├── test_login.py
│   └── test_dashboard.py
└── test-registry/
    └── manual-test-cases.csv
```

---

### Layout B — Domain-first (recommended default)

One folder per business domain. Each domain is self-contained: its own pages, tests, fixtures, and data. The `framework/` layer holds only code shared across two or more domains.

```
ui-tests/
├── pytest.ini
├── conftest.py              # global fixtures only: base_url, browser config, env
├── .env.example
├── requirements.txt
├── discovered-flows.md
│
├── framework/               # shared layer — nothing domain-specific lives here
│   ├── __init__.py
│   ├── base_page.py         # BasePage all Page Objects inherit from
│   ├── components/          # reusable UI widgets found across domains
│   │   ├── __init__.py
│   │   ├── data_table.py    # sortable/filterable table abstraction
│   │   ├── dialog.py        # confirmation/form dialogs
│   │   ├── navigation.py    # top nav / sidebar
│   │   ├── toast.py         # notification toasts
│   │   ├── date_picker.py
│   │   ├── file_upload.py
│   │   └── pagination.py
│   ├── fixtures/
│   │   ├── __init__.py
│   │   ├── auth.py          # storage_state auth fixtures, role-based login helpers
│   │   └── data.py          # shared data factory base classes
│   └── utils/
│       ├── __init__.py
│       ├── test_data_factory.py  # uuid/timestamp-based unique value generators
│       └── assertions.py    # custom soft-assert helpers, response matchers
│
├── modules/                 # one sub-folder per business domain
│   │
│   ├── auth/
│   │   ├── __init__.py
│   │   ├── conftest.py      # auth-module fixtures (pre-authenticated sessions, etc.)
│   │   ├── pages/
│   │   │   ├── __init__.py
│   │   │   ├── login_page.py
│   │   │   ├── forgot_password_page.py
│   │   │   └── mfa_page.py
│   │   ├── tests/
│   │   │   ├── test_login.py
│   │   │   ├── test_logout.py
│   │   │   └── test_password_reset.py
│   │   └── data/
│   │       └── auth_data.py
│   │
│   ├── <domain_2>/          # replace with actual domain name discovered in Phase 1
│   │   ├── __init__.py
│   │   ├── conftest.py
│   │   ├── pages/
│   │   ├── tests/
│   │   └── data/
│   │
│   └── <domain_N>/          # add one per discovered domain; no upper limit
│       └── ...
│
└── test-registry/
    └── manual-test-cases.csv
```

**How to name domains:** use the navigation labels or primary menu items from the app (e.g. Orders, Customers, Inventory, Reports, Settings, Billing). If the app has no navigation labels yet, use the noun of the primary entity managed by that area.

---

### Layout C — Domain + Role hybrid (large enterprise / multi-role apps)

Use this when the same domain area (e.g. "Orders") has meaningfully different pages and flows for different user roles (Admin vs. Customer vs. Agent). Add a role sub-layer only inside domains that actually diverge per role — do not add it everywhere.

```
ui-tests/
├── pytest.ini
├── conftest.py
├── .env.example
├── requirements.txt
├── discovered-flows.md
│
├── framework/               # same shared layer as Layout B
│   ├── base_page.py
│   ├── components/
│   ├── fixtures/
│   │   ├── auth.py          # role-aware auth: admin_page, agent_page, customer_page
│   │   └── data.py
│   └── utils/
│
├── modules/
│   ├── auth/                # auth is role-agnostic; no role sub-layer needed
│   │   ├── pages/
│   │   └── tests/
│   │
│   ├── orders/
│   │   ├── conftest.py
│   │   ├── shared/          # pages/flows identical across roles go here
│   │   │   └── pages/
│   │   │       └── order_confirmation_page.py
│   │   ├── admin/           # admin-specific pages and tests
│   │   │   ├── pages/
│   │   │   │   ├── order_management_page.py
│   │   │   │   └── bulk_action_page.py
│   │   │   └── tests/
│   │   │       ├── test_order_management.py
│   │   │       └── test_bulk_export.py
│   │   └── customer/        # customer-specific pages and tests
│   │       ├── pages/
│   │       │   ├── order_history_page.py
│   │       │   └── reorder_page.py
│   │       └── tests/
│   │           ├── test_order_history.py
│   │           └── test_reorder_flow.py
│   │
│   └── <other_domain>/
│       └── ...
│
└── test-registry/
    └── manual-test-cases.csv
```

---

### Layout D — Product + Domain (super-apps / multi-product suites)

Use when the app is a suite of independent products sharing a login and navigation shell but otherwise operating as separate entities.

```
ui-tests/
├── pytest.ini
├── conftest.py              # global: auth shell, base_url
├── .env.example
├── requirements.txt
├── discovered-flows.md
│
├── framework/               # cross-product shared layer
│   ├── base_page.py
│   ├── components/
│   ├── fixtures/
│   └── utils/
│
├── products/                # one folder per product / sub-app
│   ├── <product_a>/
│   │   ├── conftest.py
│   │   └── modules/         # domain structure inside each product (same as Layout B)
│   │       ├── orders/
│   │       ├── customers/
│   │       └── reports/
│   │
│   └── <product_b>/
│       ├── conftest.py
│       └── modules/
│
└── test-registry/
    └── manual-test-cases.csv
```

---

## Step 3 — Apply These Rules Regardless of Layout

### conftest.py hierarchy

Use pytest's native conftest discovery: a `conftest.py` at each level supplies fixtures to everything below it.

| Level | What goes here |
|---|---|
| `ui-tests/conftest.py` | `base_url`, browser config, `load_dotenv()`, environment-level auth state loading |
| `modules/<domain>/conftest.py` | Domain-specific fixtures: pre-seeded records, domain-level auth sessions |
| `modules/<domain>/<role>/conftest.py` | Role-specific auth fixtures, role-specific test data |

Never put domain-specific fixtures in the root `conftest.py`. Never duplicate a fixture that already exists at a higher level.

```python
# ui-tests/conftest.py  — global only
import os
import pytest
from dotenv import load_dotenv

load_dotenv()

BASE_URL = os.environ.get("BASE_URL", "http://localhost:3000")

@pytest.fixture(scope="session")
def base_url():
    return BASE_URL

@pytest.fixture(autouse=True)
def configure_timeouts(page):
    page.set_default_timeout(10_000)
```

```python
# modules/auth/conftest.py  — auth-module fixtures
import os
import pytest
from framework.fixtures.auth import save_auth_state

@pytest.fixture(scope="session")
def admin_auth_state(playwright, base_url):
    """Log in as admin once per session; save storage state for reuse."""
    return save_auth_state(playwright, base_url,
                           os.environ["ADMIN_USERNAME"],
                           os.environ["ADMIN_PASSWORD"],
                           storage_path=".auth/admin.json")

@pytest.fixture
def admin_page(browser, admin_auth_state):
    """Return a page pre-authenticated as admin using saved storage state."""
    context = browser.new_context(storage_state=admin_auth_state)
    page = context.new_page()
    yield page
    context.close()
```

### framework/fixtures/auth.py — storage state helper

```python
from playwright.sync_api import Playwright
from pages.login_page import LoginPage   # adjust import to actual login page path

def save_auth_state(playwright: Playwright, base_url: str,
                    username: str, password: str,
                    storage_path: str) -> str:
    """Log in once, save browser storage state to disk, return the path.
    
    Reuse the saved state across the session via browser.new_context(storage_state=...).
    """
    browser = playwright.chromium.launch()
    context = browser.new_context()
    page = context.new_page()
    login = LoginPage(page, base_url)
    login.goto().login(username, password)
    context.storage_state(path=storage_path)
    browser.close()
    return storage_path
```

### framework/base_page.py

```python
class BasePage:
    path: str = "/"

    def __init__(self, page, base_url: str):
        self.page = page
        self.base_url = base_url

    def goto(self):
        self.page.goto(f"{self.base_url}{self.path}")
        return self

    def wait_for_load(self):
        # Secondary helper — always pair with a UI-signal assertion.
        # networkidle can be unreliable on apps with polling or WebSockets.
        self.page.wait_for_load_state("networkidle")
        return self
```

### framework/utils/test_data_factory.py

```python
import uuid
from datetime import datetime

def unique_email(prefix: str = "user") -> str:
    return f"{prefix}+{uuid.uuid4().hex[:8]}@example.com"

def unique_name(base: str = "Test User") -> str:
    return f"{base} {datetime.now().strftime('%H%M%S%f')}"
```

### pytest.ini

```ini
[pytest]
addopts = --browser chromium --tracing retain-on-failure --screenshot only-on-failure -v
testpaths = modules
markers =
    smoke:   critical-path tests that must pass on every deploy
    slow:    long-running flows
    admin:   tests requiring admin role
    api:     tests with API setup/teardown
```

### .env.example

```
BASE_URL=http://localhost:3000

# Auth — provide one set per role required by the app
ADMIN_USERNAME=
ADMIN_PASSWORD=
USER_USERNAME=
USER_PASSWORD=

# Third-party / sandbox keys (if applicable)
PAYMENT_SANDBOX_KEY=
```

### requirements.txt

```
pytest
pytest-playwright
python-dotenv
pytest-rerunfailures
pytest-randomly
pytest-xdist          # parallel execution across modules
pytest-html           # HTML report generation
```

---

## test-registry/manual-test-cases.csv — format

Create this file with headers at the start of Phase 2. Append one row per test function **as each test is written** during Phase 3 — never batch-generate at the end.

### Columns

| Column | Description |
|---|---|
| `flow_name` | Short human-readable flow name matching `discovered-flows.md` |
| `test_case_id` | Unique ID, e.g. `TC-001` — increment per row |
| `domain` | Business domain / module (e.g. `orders`, `auth`) |
| `role` | User role under test (e.g. `admin`, `customer`, `anonymous`) |
| `test_scenario` | One sentence: what this specific test validates |
| `test_type` | `happy_path` or `negative` |
| `preconditions` | What must be true before the test starts |
| `test_steps` | Numbered plain-English steps separated by ` \| ` — writable for manual execution without looking at code |
| `test_data` | Inputs used (e.g. `email=<unique>; amount=100`) |
| `expected_result` | Observable outcome that proves the test passed |
| `pytest_markers` | Comma-separated markers applied (e.g. `smoke,admin`) |
| `test_file` | Relative path (e.g. `modules/orders/admin/tests/test_order_management.py`) |
| `test_function` | Exact function name |
| `automated` | `yes` for automated rows; `no` for intentionally skipped flows |
| `notes` | Flakiness flags, linked bugs, skip reasons, out-of-scope explanations |

### Example rows

```csv
flow_name,test_case_id,domain,role,test_scenario,test_type,preconditions,test_steps,test_data,expected_result,pytest_markers,test_file,test_function,automated,notes
User Login,TC-001,auth,any,Valid credentials redirect to dashboard,happy_path,User is logged out; account exists,1. Navigate to /login | 2. Enter valid username | 3. Enter valid password | 4. Click Log in,username=test@example.com; password=<TEST_PASSWORD>,Redirected to /dashboard; Dashboard heading visible,smoke,modules/auth/tests/test_login.py,test_valid_login_redirects_to_dashboard,yes,
User Login,TC-002,auth,any,Empty fields show validation errors,negative,User is logged out,1. Navigate to /login | 2. Leave all fields blank | 3. Click Log in,,Username is required error visible,,modules/auth/tests/test_login.py,test_invalid_login_shows_error[empty-empty],yes,
Order Management,TC-003,orders,admin,Admin can bulk-export selected orders as CSV,happy_path,Logged in as admin; at least 2 orders exist,1. Navigate to /orders | 2. Select 2 orders via checkbox | 3. Click Export | 4. Choose CSV format | 5. Confirm download,,CSV file downloaded containing selected order IDs,smoke,modules/orders/admin/tests/test_bulk_export.py,test_bulk_export_selected_orders_csv,yes,
Real payment submit,TC-004,checkout,customer,Complete payment with real card,happy_path,No sandbox available,,,,,,,no,Skipped — no payment sandbox. Test covers flow up to payment form only (TC-005).
```

### Rules

- One row per parametrized variant — each `@pytest.mark.parametrize` combination gets its own `test_case_id`.
- `test_steps` must allow a human QA engineer to execute the test without reading the code.
- `expected_result` must be observable — describe what a tester can see or measure.
- Skipped / out-of-scope flows still get a row with `automated=no` and an explanation in `notes`.
