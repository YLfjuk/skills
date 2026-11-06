# Tests

- Scope: Check meaningful behavior coverage, edge cases, false assumptions, isolation, and failure diagnostics. Judge tests by the guarantees they provide and the regressions they detect, rather than their count or coverage percentage alone. Follow the actual framework and repository conventions.
- Skills: Applicable installed testing and conventions skills.

## Assess test value

Identify the intended contract, meaningful failure modes, and existing coverage before proposing tests. Choose examples, properties, type tests, contract tests, or integration tests according to the guarantee needed. A missing test is actionable when a concrete regression could escape existing coverage; avoid requesting tests for every implementation branch or minor edit.

Prefer observable outcomes over assertions that mirror production logic or internal calls. Interaction assertions are useful when the interaction itself is the contract, such as avoiding a duplicate external write. Look for redundant cases, arbitrary input lists, and fixtures that obscure the behavior under test. Recommend shared fixtures or factories only when they make setup clearer. Consider property-based testing when a meaningful invariant across many inputs warrants it, without requiring a new dependency for a few useful examples.

For a failing test, determine whether the implementation, expectation, or environment is wrong by checking the intended contract. Do not recommend weakening assertions, skipping tests, or updating snapshots merely to make the suite pass.

## Type tests

Focus on type relationships and API guarantees: generic inference, conditional types, overload resolution, narrowing, constraints, preservation of literals, tuples and readonly modifiers, and unintended exposure of internal types. Prioritize the behaviors relevant to the changed API rather than requiring every category.

Before requesting a type test, name a specific change that would still compile against the implementation and existing callers but break intended consumer types or inference, such as widening an inferred return to `unknown` or merging overloads into a union signature. If existing compiler checks already reject that change, the test adds no protection against it. Checking that a function declared to return `string` returns `string` usually repeats an existing guarantee. A test of an exported signature can be useful when a consumer-breaking change, such as making an optional argument required, would otherwise compile unnoticed. Do not request signature tests routinely or flag redundant assertions without a concrete maintenance or diagnostic cost.

Negative type cases should prove that unsupported inputs or combinations fail compilation because the contract requires rejection. Keep rejected expressions minimal and check that they fail for the intended reason; an unrelated error can also satisfy `@ts-expect-error`.

Verify that the repository's type-checking command actually checks the type tests and negative assertions; a passing runtime test run alone does not establish this. Compile-time rejection also does not prove that malformed JavaScript inputs receive correct runtime handling. Request runtime validation tests when that handling is part of the contract.

## Framework features and diagnostics

Consider `.each` or the framework's equivalent for repeated scenarios with shared setup and assertions. Select meaningful equivalence classes and boundaries first; parameterization does not justify generating many redundant cases. Keep case names informative, and keep separate tests when their behavior or setup differs enough to read more clearly that way.

Consider soft assertions such as `expect.soft` when independent checks benefit from reporting all failures and remain safe to evaluate after an earlier failure. Keep prerequisite checks fail-fast, especially before dereferencing a possibly missing value. Separate tests may give clearer diagnostics for distinct behavioral requirements. Do not require soft assertions merely because a test has multiple expects.

Parameterization reduces duplication across scenarios; soft assertions improve diagnostics within a scenario. Use either or both only where the installed framework supports them and they improve the tests. Treat these as review aids, not mandatory style findings; tie recommendations to a concrete coverage, maintenance, or diagnostic problem.
