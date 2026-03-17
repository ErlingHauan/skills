---
name: studio-code-quality
description: >
  Altinn Studio code quality guidelines enforcement. Use this skill whenever writing new code,
  editing existing code, reviewing code, refactoring, or when the user asks about code quality,
  best practices, or clean code in this project. Covers function design, naming, parameters, comments, testing, and
  code organization.
---

# Studio Code Quality Guidelines

These guidelines are the team's shared standard for writing readable, maintainable code. Readability is the top priority — we spend far more time reading code than writing it. Code isn't done just because it works. Working code is the starting point; the real craft is making it readable and well-structured.

## Core Attitudes

- **Refactor proactively.** When new functionality doesn't fit the existing structure, restructure the code to accommodate it cleanly. Taking shortcuts creates technical debt that compounds over time.
- **Prefer simplicity.** Choose the simplest solution that meets the acceptance criteria. Every line of complexity costs the entire team in review, testing, and maintenance. Take pride in the code you didn't have to write.
- **Solve one thing at a time.** Keep changes small and focused — easier to review, test, debug, and less likely to cause merge conflicts.

## Function Design

### Single Responsibility Principle (SRP)

Each function does one thing at one level of abstraction. When you add a conditional or loop, extract the inner logic into its own function.

**Too many responsibilities:**
```typescript
function processOrder(order: Order) {
  // validates, calculates total, applies discount, saves to DB, sends email
  // ... 50 lines doing all of these things
}
```

**Separated by responsibility:**
```typescript
function processOrder(order: Order) {
  const validatedOrder = validateOrder(order);
  const pricedOrder = calculateTotal(validatedOrder);
  const discountedOrder = applyDiscount(pricedOrder);
  saveOrder(discountedOrder);
  sendOrderConfirmation(discountedOrder);
}
```

### Parameters

Minimize parameters — each one increases cognitive load and test combinations.

- **Group related parameters** into an object.
- **Split the function** — many parameters often signal too broad a responsibility.
- **Avoid boolean flags** — they make call sites unreadable (`renderComponent(data, true, false)`). Create two separate functions instead: `renderEditableComponent(data)` and `renderReadOnlyComponent(data)`.
- **Avoid optional parameters** — a parameter that's only needed sometimes signals the function handles too many cases. Split by use case: create a focused function for each scenario, composing from a shared core. This keeps each function's contract clear.

The same applies to React component props — optional props create ambiguity and special cases. Prefer splitting into distinct components for distinct use cases over adding boolean/optional props.

## Naming

Names are the primary documentation when reading code. Invest time so the code cannot be misunderstood.

- **Functions use verbs**: `fetchUserProfile`, `calculateTotal`.
- **Boolean-returning functions are questions** — prefix with `is`, `has`, `should`, `are`: `isValid`, `hasPermission`.
- **Include units** for numeric values: `debounceTimeInMilliseconds`, `maxRetryCount`.
- **Avoid abbreviations** — especially `a11y`, `i18n`, `e2e` which confuse those unfamiliar with them.
- **Use established prefixes/suffixes consistently** — e.g., `use[...]Query` for React Query hooks.
- **Long and precise beats short and vague** — a long name may signal unclear responsibility, but it's still better than an ambiguous one.

## Code Organization

### Newspaper principle

High-level functions come first, helpers afterward. This lets readers scan top-down without jumping around. In JavaScript, use `function` declarations (hoisted) rather than `const` arrow functions when defining helpers below their usage.

### Proximity

Define helpers right after the function that calls them. On the filesystem, if a file is only used by one other, place them in the same directory.

### No magic values

Hardcoded numbers and strings (other than `0`, `1`, `""`) should be assigned to descriptively named constants. Exception: when TypeScript's type system makes the value self-documenting — e.g., `mode === 'edit'` when `mode` is typed as `'edit' | 'read'`.

### Wrap external dependencies

Don't couple directly to external packages — build an abstraction layer so the rest of the codebase depends on your interface, not the third-party one. Migration then happens in one place.

## Comments

Comments go stale and we can't test them. Prefer making code self-descriptive. Only comment when necessary:

- Linking to a task/issue explaining a workaround or dependency bug.
- Explaining *why* when the reason isn't obvious from the code.
- Explaining performance-motivated deviations from the readable approach.

Never comment out code — use git history to recover old code.

## Testing

Test code gets the same quality standards as production code — readability, SRP, good naming.

### TDD cycle

1. Write a test for a basic case.
2. Implement just enough to pass.
3. Write a test for a more specific case.
4. Extend the implementation.
5. Repeat until complete.

Writing tests after risks them being shaped by the implementation rather than the requirements.

### One test per data set

Give each input combination its own test case — when a test fails, you need to know exactly which data set caused it. Use `it.each` / `test.each` in Jest to keep this clean.

### Cover all outcomes

Line coverage alone isn't sufficient — a single regex can have many outcomes. Identify the decisive factors, vary one at a time, and cover boundary values.

**Example — date validation:** Test years, then months, then days separately. For days, test 30-day and 31-day months. Check February for leap/non-leap years. If the input is a string, format is an additional factor.

### Test isolation

Reset all test data before each test. Leaking state between tests is the most common cause of flaky tests.

### Verify tests can fail

False positives are devastating. For tests asserting something should *not* happen, temporarily make the function return the failure value — confirm the test catches it.

### Unit vs E2E

Unit tests cover all code paths — verify every parameter combination produces the expected result. Focus on public API outcomes, not internal helpers. E2E tests verify integrations between components — test that a user action produces results across the entire system, not full code coverage.
