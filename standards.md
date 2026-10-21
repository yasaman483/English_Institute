# Shared Engineering Standards

These standards apply to every stage of the primary workflow (`workflow/software-engineering-workflow.md`) and to every skill and role. Skills and roles refer to this file instead of repeating these rules.

## 1. Coding and architecture

1. Keep modules small and single-purpose. Separate request handling, business rules, and data access.
2. Validate all input at the system boundary (API, form, file upload, external response). Trust no client-side validation alone.
3. Prefer the framework's built-in and well-maintained libraries over custom code for auth, crypto, parsing, and sessions. Do not write your own cryptography.
4. Use parameterized queries or the ORM's safe API. Never build SQL or shell commands by string concatenation with user data.
5. Handle errors explicitly. Return consistent error responses. Do not expose stack traces or internal details to users.
6. Keep configuration out of code. Read it from environment variables or a secret store.
7. Add a dependency only with user approval (see the Package Management rule in the project instructions). Use package managers to add or update dependencies.
8. Keep changes inside the increment's file scope. Note any necessary out-of-scope change in the implementation notes.
9. Write code comments only where the reason is not obvious. Do not comment what the code plainly says.

## 2. Requirements and identifiers

Every traceable item has a stable ID. IDs are never reused.

| Item | Format | Example |
|---|---|---|
| Functional requirement | FR-nnn | FR-012 |
| Non-functional requirement | NFR-nnn | NFR-004 |
| Acceptance criterion | AC-<req>-<letter> | AC-012-a |
| Test case | TC-<req>-<nn> | TC-012-01 |
| Architecture decision | ADR-nnn | ADR-003 |
| Increment | INC-nnn | INC-002 |
| Defect | DEF-nnn | DEF-007 |
| Risk | RSK-nnn | RSK-001 |
| Assumption | ASM-nnn | ASM-004 |

An acceptance criterion is written as a testable statement: "Given <context>, when <action>, then <observable result>." Vague criteria such as "works well" are not allowed.

Requirements, ACs, and their status live in `docs/requirements.md`. Other documents reference IDs and do not restate requirement text.

Traceability (`docs/traceability.md`) maps each AC to its test cases, increment, and latest status. Each row lists: AC ID, requirement ID, increment, test IDs, status, evidence path.

## 3. Documentation conventions

1. Markdown is the primary format. Generate Word or PDF only when the user asks.
2. Each document starts with a title, a one-line purpose, and a "Last updated" date.
3. Use tables for IDs and statuses. Use prose for decisions and rationale.
4. Record every assumption in `docs/assumptions.md` with its consequence if wrong.
5. Record every decision with a context, the options considered, the decision, and its consequences in an ADR.
6. Changes to an accepted requirement are logged in `docs/requirements.md` under a change log section with date and reason.

## 4. Status vocabulary

Use exactly these statuses in test reports, traceability, and status reports.

| Status | Meaning | Requirement for use |
|---|---|---|
| PASS | Test ran and the observed result matched the expected result | Evidence (command output, log, screenshot, or recorded result) must be attached |
| FAIL | Test ran and the observed result did not match | Defect record must exist |
| BLOCKED | Test could not run because of a dependency, missing tool, or missing credential | Blocker must be named |
| NOT RUN | Test is planned but has not been executed | Used for planned tests; never shown as PASS |
| NOT APPLICABLE | Test type does not apply to this change | Reason must be stated |

Rules:
1. Missing evidence is never PASS.
2. Planned or described tests are NOT RUN until executed.
3. A test that cannot be executed in the current environment is BLOCKED or NOT RUN, with the reason. It is not silently dropped.
4. A claim that code works is not evidence. Only executed results, reviewed output, or a recorded manual observation count.
5. An increment is DONE only when every AC it covers is PASS or has an accepted, documented exception.

Increment status uses: NOT STARTED, IN PROGRESS, IN REVIEW, REWORK, DONE, DEFERRED.

## 5. Defect record

Every defect has one record (in `docs/defects/` or the defect log). Fields:

- Defect ID, title, date found, found by (role).
- Linked AC and requirement IDs.
- Severity: critical (security, data loss, blocks main journey), major (feature broken, workaround hard), minor (cosmetic or edge case).
- Steps to reproduce, expected result, actual result.
- Evidence (test report path, log excerpt with secrets removed, screenshot path).
- Status: OPEN, IN FIX, FIXED (awaiting verify), VERIFIED, CLOSED, WONTFIX (with accepted reason).
- Fix summary, files changed, retest IDs, verification evidence.

A defect moves to VERIFIED only after QA reruns the failing test and the regression set and records PASS.

## 6. Risk and assumption records

Risk: ID, description, likelihood (low, medium, high), impact (low, medium, high), mitigation, owner, status.

Assumption: ID, description, reason it was needed, consequence if wrong, how to verify, status (OPEN, CONFIRMED, INVALID).

## 7. Security and secret management

1. Never place secret values in code, documents, test reports, logs, tickets, or chat messages. Refer to secrets by name only (for example `DATABASE_URL`).
2. Never print environment variables, config files containing secrets, credential files, auth tokens, cookies, or session files. Verify presence by name or count.
3. Use `.env.example` with placeholder values for documentation. Keep real `.env` files out of version control; check `.gitignore` before the first commit.
4. Passwords are hashed with a modern adaptive algorithm supported by the chosen framework. Session and token handling follows the framework's defaults unless an ADR states otherwise.
5. Authorization is checked on the server for every protected operation, not only in the UI.
6. Logs must not contain passwords, tokens, full personal data, or payment data.
7. Dependencies with known critical vulnerabilities are reported to the user before use.
8. Any step that publishes a site, changes production data, or incurs cost requires user authorization (see workflow section 8).

## 8. Accessibility baseline

Unless the requirements state otherwise, target WCAG 2.2 Level AA for user-facing pages. Keyboard operation, visible focus, text alternatives for images, labelled form fields, sufficient colour contrast, and meaningful page titles are required. Accessibility acceptance criteria are checked in system-level verification (workflow Stage 8).

## 9. Performance and reliability baseline

When the requirements do not give budgets, record these defaults as assumptions: first contentful paint under 2.5 seconds on a typical mobile profile, API responses under 500 ms at p95 for simple reads at the expected load, and graceful error pages for every failed request path. Replace these with measured requirements when the user provides them.

## 10. Quality gates (per increment)

An increment passes its quality gate when:
1. Code builds without errors.
2. Lint and type checks (where the stack provides them) report no new errors.
3. Applicable tests from the test plan are PASS or accepted exceptions.
4. No open critical or major defect remains in the increment.
5. Security-relevant changes have their security checks executed or recorded as BLOCKED with a reason.
6. Traceability is updated.

Commands for build, lint, and test are recorded in the project's README and in each test report. Do not invent commands; discover them from the project's configuration files and record what was actually run.
