# External documentation

Use this reference for documentation intended for users of the project, including READMEs, guides, CLI documentation, and interactive examples.

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

## Tone and review

Use direct, practical language without marketing or unnecessary implementation details.

Apply the shared review in [SKILL.md](../SKILL.md), then read as a new user:

* Can I understand the product and complete a common task without excessive reading?
* Are useful workflows, troubleshooting, and advanced options easy to find?
* Do examples work with the stated setup?
* Is maintainer-only material in a more appropriate location?
* Do guides link to reference rather than copy it?
