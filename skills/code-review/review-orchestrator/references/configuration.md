# Configuration and personal extensions

Accept natural-language configuration; do not make the user fill in a form. Use these defaults for omitted settings:

| Setting | Default |
| --- | --- |
| Review URL | User-specified full PR/MR URL, or an unambiguous URL from reliable session context |
| Host/API endpoint | Infer from the review URL and available host connection; accept an explicit endpoint when needed |
| Repository workspace | Current workspace when it matches the target; accept an explicit local or remote workspace |
| Maximum reviewers | `auto`, choose the smallest useful set without a numeric cap |
| Reviewer plan | Select instances based on the change, unless the user supplies a plan |
| Execution mode | `subagents` |
| Default model | `inherit`, the orchestrator's current provider and model |
| Model selection policy | `preferred` |
| Per-role model/policy | Omitted fields inherit the default |
| Per-instance model/policy | Omitted fields inherit the role settings |
| Additional instructions | None |
| Role extension file | Explicit path, otherwise `~/.config/review-orchestrator/roles.md` |

## Review location

Preserve the configured review URL's hostname, port, and path. A request number alone is sufficient only when the host and repository are unambiguous.

Keep the review URL, host/API endpoint, and repository workspace separate. Use the host connection for discussions and request state, and the workspace for code inspection and verification. Pass these resolved settings to every reviewer. Confirm that the environment can access the host and that the selected delegation mechanism can use the intended workspace. Report unavailable access and ask for the missing connection or a supported alternative.

## Reviewer instances and limits

A role defines responsibilities and skills. An instance is one reviewer assigned that role. Accept repeated roles with identical or different models, including cross-provider choices when available. Assign a unique stable instance ID such as `correctness-1` to each reviewer and retain it across follow-up rounds. An explicitly repeated entry requests another reviewer; do not deduplicate instances by role, scope, provider, or model. Reusing an existing instance ID updates that instance rather than launching another. Ask if duplicate IDs make the intended plan ambiguous.

A numeric maximum limits the size of the reviewer plan, not the number of reviews each instance may perform. Accept `Reviewer count` as an alias for this maximum. Choose fewer reviewers when sufficient; never fill unused slots just because they exist. With `auto` and an explicit plan, use its listed instances. Without an explicit plan, choose the smallest useful set.

For example, this configuration permits up to four reviewers but requests only two:

```markdown
Maximum reviewers: 4
Reviewers:
- ID: correctness-1
  Role: Correctness
  Model: inherit
- ID: correctness-2
  Role: Correctness
  Model: {available provider/model}
  Model selection policy: exact
```

Accept natural-language requests too, such as "two Correctness reviewers on my current model and one on this other model." Keep explicitly requested instances within the maximum. If the supplied plan exceeds the cap, ask which instances to remove or whether to raise the cap before launching. Approval of a new role or saving it to the extension file does not raise the cap. Recommend extra coverage separately when it would exceed the cap. Replace or retire instances deliberately and preserve their review history.

## Model choices

Discover the models and providers available through the current environment's supported capabilities. Use actual identifiers returned by those capabilities when required by launch tools. Do not infer availability from a model name in a prompt or a skill file.

Accept per-role and per-instance model choices. Resolve each field in this order: explicit instance setting, explicit run-level role setting, personal role setting, run default. Resolve each field independently. An explicit `inherit` selects the orchestrator's current provider and model, even when the default is another model. Instance scopes and additional instructions can narrow or supplement their role definition without editing it.

For `exact`, never substitute without asking. For `preferred`, use the requested model if available; otherwise choose an available alternative and explain the substitution. For `auto`, choose from available models and briefly explain why. If capability discovery is unavailable, use only model choices supported by the environment and state the selection limit.

## Update-safe role file

Read the explicitly configured extension file, otherwise `~/.config/review-orchestrator/roles.md`. Resolve `~` to the current user's home directory on the host running the orchestrator. This path is independent of the agent provider. Keep it outside the installed skill directory so replacing the skill does not overwrite personal roles. Preserve existing content during updates.

A missing file means bundled roles apply. Do not create one just to run a review. When the user asks to save a role or configure its skills, append or amend the requested block, preserving other content. A direct request to save a supplied block authorizes that edit. Approval to use a role for one run does not authorize saving it permanently.

Role headings contain only the role name. Entries in this file are user-approved and eligible for selection:

```markdown
### Accessibility
- Scope: Check affected interfaces for keyboard access, focus handling, and accessible names.
- Skills: Relevant installed accessibility-review skills.
- Model: inherit
- Model selection policy: preferred
- Additional instructions: Support concerns with a concrete user interaction.
```

A heading matching a bundled role overrides its explicitly supplied fields. A new heading adds a role. To supplement a role without copying its scope:

```markdown
### Docs
- Additional skills: my-team-docs
- Additional instructions: Check examples against the public CLI.
```

Supported fields are `Scope`, `Skills`, `Additional skills`, `Model`, `Model selection policy`, and `Additional instructions`. Roles may include further Markdown instructions under their heading, including procedures and metadata requirements. A role that needs additional inputs declares what to collect, its defaults, reliable inference sources, and when to ask the user. Role-specific settings belong with that role. Include its instructions and collected metadata in selected reviewer briefs. `Skills` replaces the configured skill list; `Additional skills` appends to it. Omitted fields retain existing values. Apply repeated blocks in file order, with later explicit fields winning and additional skills deduplicated.

Normalize a trailing `Reviewer` and case when matching role names. Clarify ambiguous matches or unusable fields for the affected role; independent roles may proceed. Role matching must not collapse distinct reviewer instances. Extensions configure roles, not execution ownership, host permissions, or invocation policy. Saved roles do not launch automatically or increase the cap.

## Proposing a role during a review

Explain its purpose, the specific risk, suggested model, and proposed number of instances. Provide a copyable block with concrete values:

```markdown
### {Role}
- Scope: {specific responsibilities}
- Skills: {relevant installed skills, if any}
- Model: {inherit, or provider/model}
- Model selection policy: {exact | preferred | auto}
- Additional instructions: {instructions, if any}
```

Ask whether to use the role for this run and whether to save it. If the proposed plan exceeds the cap, ask for a cap increase or replacement as a separate decision. Existing authorized work can continue while that decision is pending. Do not interpret silence as approval. Keep instance counts and IDs in run configuration, not in saved role definitions.
