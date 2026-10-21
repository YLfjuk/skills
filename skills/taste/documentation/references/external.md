# External documentation

Use this reference for documentation intended for users of the project, including READMEs, guides, release notes, CLI documentation, and interactive examples.

## Organize around user tasks

Help readers understand what the product does, start using it, complete common tasks, discover important options, and troubleshoot failures. Do not assume familiarity with the source or internal architecture.

Group information around goals and workflows. Do not mechanically create one page per source file, internal module, class, feature, or command. A separate page should help users find or complete something.

Introduce a concept with a short explanation and a concrete example. Follow with important variations, then link to detailed reference as needed. Keep advanced functionality discoverable without forcing new users through it.

## Separate guides from reference

Guides explain when a feature is useful and how to complete a task. Reference records exact syntax, supported values, constraints, and complete interfaces.

Reference may be exhaustive; a getting-started guide should not be. Link to precise details rather than reproducing the full reference in conceptual pages.

For a programmatic API, use guides for realistic integration examples and generated reference for signatures, types, parameters, and overloads. See [API documentation](api.md) when working on that reference.

## Use practical examples

Prefer realistic, copyable examples that accomplish a task. Include the prerequisites or setup needed to make them work. Avoid examples designed only to demonstrate every option combination, and cut surrounding prose when the example already explains the behavior.

For a CLI, prioritize installation, common commands, important options, and troubleshooting. Group advanced flags in reference rather than filling the primary guide with every flag.

When the project provides a browser runner, playground, or terminal, make examples easy to transfer into it. Help readers understand an example, try it, modify it, and explore further. Documentation and the interactive tool should complement each other without duplicating the entire experience.

## Package release notes

Check how the project's changelog tooling distributes each note. Do not assume package headings restrict which changelogs receive the body. If the tool sends the complete body to every package named in frontmatter, review that whole entry for each recipient.

For each recipient, identify the consumer-visible change and check that the entire entry applies to that package's users. Split independent package-specific API, command, installation, or dependency changes. Keep shared compatibility requirements, producer/consumer contracts, and coordinated migrations together when the full explanation helps each audience. Mention another package only where it helps explain the recipient's change.

Compare the note with the last published behavior and the package that owns the changed functionality at release time. Check both missing and excess attribution: affected packages need their changes documented, and every recipient needs an applicable entry. Shared version groups or dependency relationships alone do not establish relevance.

When reviewing, report the concrete note and mismatched claims, with proposed package splits where needed.

## Tone and review

Use direct, practical language without marketing or unnecessary implementation details.

Apply the shared review in [SKILL.md](../SKILL.md), then check the relevant document type:

* Can a guide's intended reader complete the task with the stated prerequisites?
* Are useful workflows, troubleshooting, and advanced options easy to find?
* Do examples work with the stated setup?
* Is maintainer-only material in a more appropriate location?
* Do guides link to reference rather than copy it?
* Does each release-note recipient receive an applicable, complete account of its changes and required migration steps?
