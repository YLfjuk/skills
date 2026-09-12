# External Documentation

Use this reference when creating or modifying documentation intended for users of the project.

## Audience

Write for someone who wants to **use the product**, not someone who wants to understand its implementation.

The documentation should answer questions such as:

* What is this?
* Why would I use it?
* How do I get started?
* How do I accomplish common tasks?
* What are the important options?
* What should I do when something goes wrong?
* Where can I find detailed reference information?

Do not assume the reader is familiar with the source code or internal architecture.

## User-focused documentation

Organize documentation around the user's goals and workflows rather than the structure of the implementation.

Do not mechanically create:

* One page per source file
* One page per internal module
* One page per class
* One page per feature simply because it exists
* One page per command without considering how users actually discover and use it

Instead, group related functionality around meaningful user tasks.

## Progressive disclosure

Introduce complexity gradually.

Prefer:

1. A short explanation
2. A concrete example
3. The most important options or variations
4. Detailed reference information when necessary

A new user should be able to get started without reading the entire documentation.

Advanced functionality should be discoverable without overwhelming the primary learning path.

## Examples

Prefer realistic, copyable examples over lengthy explanations.

Examples should demonstrate how users actually use the product.

Do not create examples merely to demonstrate every possible option combination.

When an example is sufficient to explain behavior, avoid surrounding it with unnecessary prose.

## Reference vs guides

Keep conceptual and task-oriented documentation separate from exhaustive reference material.

### Guides

Explain:

* Why something is useful
* When to use it
* How to accomplish a task
* Common workflows
* Recommended approaches

### Reference

Explain:

* Exact syntax
* Available options
* Supported values
* Constraints
* Complete public interfaces

Reference material may be more comprehensive, but it should still be organized and readable.

## CLI documentation

For CLI projects, prioritize the user's workflow.

The primary documentation should generally make it easy to:

* Understand the CLI
* Install or start using it
* Run the most common commands
* Understand important options
* Discover advanced functionality
* Troubleshoot common problems

Do not turn the primary documentation into an exhaustive dump of every flag.

If the CLI has many options, prioritize commonly used options and group advanced options appropriately.

## Browser-based runners

If the project provides a browser-based runner, playground, terminal, or similar interactive experience, treat it as part of the documentation experience.

Documentation and the interactive tool should complement each other.

Prefer a flow such as:

**Understand → Try → Modify → Explore**

Do not duplicate the interactive experience unnecessarily inside the documentation.

Where appropriate, make examples easy to transfer into the interactive environment.

## Programmatic APIs

When documenting a programmatic API, distinguish between:

**Conceptual documentation**

Explains when and why a developer would use the API and provides useful examples.

**API reference**

Provides the complete public API surface, signatures, types, parameters, and other precise details.

Do not copy the generated API reference into conceptual documentation.

A conceptual page should link to the appropriate API reference when detailed information is needed.

## Tone

Use a tone that is:

* Clear
* Direct
* Professional
* Approachable
* Practical
* Confident

Avoid:

* Marketing-heavy prose
* Excessive enthusiasm
* Filler
* Repetition
* Unnecessary jargon
* Walls of text
* Artificially formal language

The documentation should feel curated and intentional rather than automatically generated.

## Quality bar

Review the documentation as a new user.

Ask:

* Can I understand the product quickly?
* Can I accomplish a common task without excessive reading?
* Are the most important workflows easy to find?
* Are examples realistic and useful?
* Is advanced functionality appropriately separated?
* Is anything primarily useful to maintainers rather than users?
* Could any page be significantly shorter?
* Is there duplicated information that belongs in a reference instead?
