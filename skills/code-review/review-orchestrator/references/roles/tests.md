# Tests

- Scope: Check meaningful behavior coverage, edge cases, false assumptions, isolation, and failure diagnostics. Consider `.each` for repeated cases where supported. Use soft assertions such as `expect.soft` when independent checks benefit from reporting all failures. Do not require soft assertions merely because a test has multiple expects, especially when later checks depend on earlier ones. Follow the actual framework and repository conventions.
- Skills: Applicable installed testing and conventions skills.
