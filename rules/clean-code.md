# Clean Code Guidelines

Write code following Clean Code principles. Optimize for readability, simplicity, maintainability, and clear intent.

## Core Principles

- Prefer simple, explicit code over clever or overly abstract solutions.
- Keep functions and methods small and focused on one responsibility.
- Use meaningful names for variables, functions, classes, modules, and parameters.
- Avoid unnecessary abstractions, indirection, wrappers, and helper functions.
- Avoid premature generalization. Implement the simplest design that satisfies the current requirement.
- Reduce duplication when doing so improves clarity, but do not create abstractions solely to eliminate a few repeated lines.
- Prefer early returns and guard clauses over deeply nested conditionals.
- Keep business logic separate from infrastructure, I/O, framework glue, and presentation logic where practical.
- Preserve existing architectural conventions unless there is a strong reason to change them.

## Comments

Treat comments as a last resort, not as documentation for otherwise readable code.

Code should explain WHAT it does. Comments should explain WHY.

Do not write comments that merely restate the code. Avoid comments such as:

```python
# Get the user
user = get_user(user_id)

# Check if user is active
if user.active:

# Loop through orders
for order in orders:

# Return the result
return result
```

Instead, make the code itself expressive through good naming and structure.

Before adding a comment, ask:

1. Can clearer naming remove the need for this comment?
2. Can the code be simplified?
3. Can this logic be extracted into a well-named function?
4. Can the structure make the intent obvious?

If yes, improve the code instead of adding the comment.

Comments are appropriate when they explain:

- Why a non-obvious implementation was chosen.
- Business rules that cannot reasonably be inferred from the code.
- External system limitations or workarounds.
- Important side effects or invariants.
- Security or concurrency constraints.
- Compatibility issues.
- A surprising decision that a future developer may otherwise "fix" incorrectly.

Example:

```python
# Odoo may retry this webhook, so the operation must remain idempotent.
process_payment_once(payment_id)
```

Do not use comments as section headings inside ordinary functions:

```python
# Validate input
...

# Process data
...

# Save result
...
```

If a function needs section headings to be understandable, consider splitting it into smaller, well-named functions.

## Docstrings

Do not automatically add docstrings to every function or class.

For simple internal functions where the signature and name clearly communicate the behavior, omit the docstring.

Use docstrings primarily for:

- Public APIs.
- Complex contracts.
- Non-obvious parameters or return semantics.
- Important side effects.
- Exceptions that callers need to understand.

Do not write docstrings that simply repeat the function name, parameters, or implementation.

## Naming

Prefer names that reveal intent.

Avoid vague names such as `data`, `info`, `item`, `obj`, `temp`, `result`, `process_data`, `handle_item`, `do_work` when a more
domain-specific name is available.

Prefer names such as `eligible_benefits`, `invoice_lines`, `caregiver_assessment`, `calculate_discount`, `find_expired_sessions`.

Names should make comments unnecessary whenever possible.

## Functions

A function should have one clear purpose.

Prefer:

```python
eligible_users = find_eligible_users(users)
send_expiration_notifications(eligible_users)
```

over a large function containing unrelated steps separated by comments.

Avoid extracting trivial one-line functions unless the extracted name communicates meaningful domain intent.

## Error Handling

- Fail clearly rather than silently swallowing errors.
- Catch exceptions only when the code can meaningfully handle them.
- Avoid broad exception handling unless required at a system boundary.
- Error messages should contain enough context to diagnose the problem.
- Do not use comments to compensate for unclear error-handling behavior.

## Refactoring Existing Code

When modifying existing code:

- Make the smallest coherent change necessary.
- Do not rewrite unrelated code.
- Preserve existing behavior unless the task explicitly requires changing it.
- Follow the surrounding project's naming, architecture, formatting, and conventions.
- Remove obsolete comments when code changes make them inaccurate.
- Do not introduce comments just because you modified a section.

## Agent Behavior

When generating or modifying code:

1. Understand the surrounding code and conventions first.
2. Implement the simplest correct solution.
3. Refactor the implementation until its intent is understandable from the code itself.
4. Add comments only when important reasoning still cannot be expressed clearly through code.
5. Remove redundant comments generated during implementation.
6. Before finishing, review every new comment and ask: "Would this comment still be useful to an experienced developer who can
   already read the code?" If not, remove it.

The desired result is code that can normally be understood without comments, with a small number of high-value comments
explaining decisions, constraints, or non-obvious reasoning.
