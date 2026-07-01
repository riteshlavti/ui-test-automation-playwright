# Playwright QA Automation Agent

An autonomous Playwright QA Automation skill that explores a running web application, discovers user journeys, asks for missing information instead of making assumptions, and generates a scalable, production-ready Playwright automation framework.

Unlike traditional code-generation prompts, this skill follows a structured QA workflow—from exploration through execution—to produce maintainable automation that scales with your application.

---

## Features

- 🔍 Automatically explores the application before writing tests
- 🗺️ Discovers pages, business modules, and user flows
- ❓ Asks for credentials, test data, or business rules instead of hallucinating
- 🧱 Scaffolds a production-ready Playwright project
- 📦 Organizes the framework by business modules for long-term maintainability
- 📄 Generates Page Objects and reusable UI components
- ✅ Creates happy-path and negative test scenarios
- ▶️ Executes the generated tests
- 🔄 Iteratively fixes automation issues where possible
- 📊 Maintains a synchronized manual test registry
- 📚 Produces flow documentation and coverage artifacts

---

## Workflow

The skill follows a structured, multi-phase QA workflow:

1. **Setup**
   - Verify application accessibility
   - Verify Playwright tooling
   - Determine authentication strategy

2. **Application Exploration**
   - Discover navigation
   - Identify business modules
   - Map user journeys
   - Record application flows

3. **Project Scaffolding**
   - Generate a modular Playwright framework
   - Create reusable architecture
   - Organize by business capability

4. **Test Generation**
   - Generate Page Objects
   - Generate reusable components
   - Create automated tests
   - Generate test data

5. **Execution & Validation**
   - Execute the suite
   - Analyze failures
   - Repair automation issues where appropriate
   - Report application defects separately

6. **Delivery**
   - Generate documentation
   - Update test registry
   - Produce coverage summary

---

## Example Prompts

### Explore an application

```
Explore https://staging.example.com and generate a Playwright automation framework.
```

---

### Generate regression suite

```
Generate a complete regression suite for my application using Playwright.
```

---

### Existing project

```
Explore my existing application and add Playwright automation without changing the current project structure.
```

---

### Specific module

```
Generate Playwright tests only for the Customer Management module.
```

---

### Existing repository

```
Use the existing Playwright framework and generate automation for newly added features.
```

---

## What the Skill Generates

Depending on the application, the generated framework may include:

- Modular project structure
- Business module organization
- Page Objects
- Reusable UI components
- Fixtures
- Test data factories
- Environment configuration
- Authentication helpers
- Automated test suites
- Flow documentation
- Test registry
- Coverage reports

The exact structure is derived from the discovered application instead of relying on a fixed template.

---

## Design Principles

This skill is designed around several core principles:

- Explore before generating code
- Never hallucinate credentials or business rules
- Organize automation by business capability
- Prefer reusable components over duplicated locators
- Keep tests independent and maintainable
- Generate production-quality automation rather than demo scripts

---

## Best Practices

The generated automation follows Playwright best practices, including:

- Accessible locators
- Page Object Model
- Reusable components
- Environment-based configuration
- Parallel execution support
- Independent tests
- Dynamic test data
- Storage-state authentication when appropriate
- Trace and screenshot capture
- Scalable project organization

---

## Requirements

- Playwright
- pytest
- pytest-playwright
- Python 3.10+
- A running web application
- Playwright MCP (recommended for live exploration)

---

## Included References

This skill includes additional reference documentation for:

- Exploration workflow
- Project organization
- Playwright best practices

These references guide the agent while generating automation and help maintain consistency across projects.

---

## Ideal Use Cases

- Building a new Playwright framework
- Creating regression suites
- Automating enterprise applications
- Expanding existing Playwright projects
- Accelerating QA automation
- AI-assisted test generation

---

## License

MIT License
