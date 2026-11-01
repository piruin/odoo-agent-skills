---
name: unit-testing-guidelines
description: Apply when writing, changing, or reviewing unit tests, including code reviews that contain unit tests. Guide test boundaries, assertions, determinism, and risk-based coverage.
---

# Unit-Testing Guidelines

Use these principles when creating or changing tests and when reviewing tests in a code change. Follow the project's test framework and conventions. In reviews, report concrete gaps in the behavior a test is meant to protect; do not require tests solely for coverage.

## Test observable behavior

- Exercise the intended public interface and assert externally observable results.
- Avoid coupling tests to private methods, internal helpers, implementation-specific call order, or internal object structure.
- Use mocks or spies when an interaction is itself part of the contract, such as sending exactly one notification or passing specific data to an external service. Do not mock the behavior under test.
- An internal refactor should not break a test if observable behavior stays the same.

## Establish the expected result

- Derive expected values from requirements, business rules, or clearly defined behavior. Be able to explain why each expected result is correct without copying production logic into the test.
- If the requirement is unclear, investigate or ask rather than inventing a rule to make a test pass.
- When capturing existing behavior before refactoring, identify the test as a characterization test; it does not prove that behavior is correct.

## Make each scenario clear and effective

- Give each test one clear scenario. Multiple assertions are appropriate when they describe the same outcome.
- Name the condition and expected behavior, such as `test_rejects_order_when_stock_is_insufficient`.
- Keep Arrange, Act, and Assert clear. Prefer readable scenario setup over excessive test abstraction; a little duplication is acceptable.
- Make assertions specific enough to detect the defect being tested. Ask: if production behavior were wrong, which assertion would fail? Avoid weak assertions when an exact result is required.
- Never assert a return value configured on a mock of the behavior under test.
- Add comments only for non-obvious business rules, constraints, or exceptions, not to narrate test steps.

## Choose a reliable test boundary

- Control time, randomness, mutable globals, network calls, and shared state so tests are independent and repeatable.
- Use fast unit tests for pure business logic and framework test facilities for behavior that depends on framework integration.
- In Odoo, use facilities such as `TransactionCase` for ORM, database, constraint, or permission behavior. Do not mock `env`, `search`, `create`, and `write` by default when testing real model behavior.
- Replace external systems with suitable test doubles at their boundary.

## Prioritize risk and preserve test integrity

- Prioritize core business rules, boundaries, invalid inputs, failure paths, reported bugs, and costly regressions. Treat coverage as a diagnostic, not a quality target.
- For a bug fix, add a regression test when practical; confirm it would fail for the original defect and pass after the fix.
- Do not make a test pass by weakening assertions, skipping or deleting it, or changing the expected result without a requirement change.
- Report what tests actually ran and what did not run.
