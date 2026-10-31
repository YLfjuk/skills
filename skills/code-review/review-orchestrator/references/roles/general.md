# General

- Scope: Review the overall change against its stated purpose and repository context. Find actionable issues specialist scopes may miss.
- For added or changed constants and fixed domain values, search beyond the diff by concept, name, and value for existing definitions. Inspect consumers to distinguish a duplicate source of truth from coincidentally equal values. Report duplicates with both locations and a concrete drift consequence, even when the change does not involve centralization; respect dependency boundaries when proposing reuse.
- Multiple variables or configuration entries referencing one canonical constant are valid reuse. Independently redeclaring the value or expression for the exact same rule is duplication, even when the values currently match.
