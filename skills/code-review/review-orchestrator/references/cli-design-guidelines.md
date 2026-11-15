<!-- Offline snapshot of Vercel's CLI Design Guidelines.
Source: https://vercel.com/design/cli
Markdown: https://vercel.com/design/cli.md
Retrieved (UTC): 2026-11-15
SHA-256 of source Markdown (UTF-8): c8f5ec44cf8bac6d5bf2157aba6b59147db67e57a0ae5b385287a95807280390
Refresh by replacing the source Markdown below and updating this metadata.
-->

# CLI Design Guidelines

A manual for building command-line interfaces that people can learn, programs can compose, and agents can operate with confidence.

## Principles

### Meet the bar

Every command answers 4 questions, for the person at the terminal and for the script that calls it:

1. **What will it act on?** The resolved team, project, and resource, shown before anything consequential happens.
2. **What will change?** Broad or risky effects, shown before they happen.
3. **What happened?** The real outcome (done, pending, no-op, partial, canceled, or failed), in the output and in the exit code.
4. **What next?** One exact, working command, when there is a next step.

The best command is fast, keeps the common path short, asks only for what it can't infer, and never guesses on consequential input. It behaves the same in every mode, writes stdout you can pipe, fails with errors you can act on, and never breaks a script silently.

### Start with a capability

A capability is something a person or program can accomplish: deploy a project, inspect a deployment, rotate a credential, recover from a failed upload. Define it first, with its inputs, permissions, effects, result, and failures. Choose syntax last. Design first use, empty results, long work, failure, cancellation, and recovery alongside success.

Every rule in this manual follows from 7 principles:

1. **Capability first.** A capability means the same thing in the CLI, API, SDK, MCP server, and dashboard.
2. **One concept, one name.** One noun per concept, one verb per operation, one option per input, all from the [Reference](https://vercel.com/design/cli#reference).
3. **Resolve before acting.** Every consequential target is exact and verified. Missing or invalid input is never a reason to guess, create, or switch targets.
4. **Interaction, consent, and format are independent.** Prompts need a TTY. `--yes` grants consent but supplies no data. `--json` selects a format and turns off prompts, but grants no consent. Piping stdout changes none of them. The one exception is compatibility: Logs and Traces Search keep their [JSONL](https://vercel.com/design/cli#output-streams-and-other-protocols) in non-interactive runs.
5. **stdout is the answer.** Results go to stdout; everything else goes to stderr. Machine formats are exact contracts.
6. **Report reality.** Report accepted work as accepted and partial work as partial, and make the exit code match the outcome.
7. **Protect people and data.** Secrets never leak into incidental output. Destructive work requires proof of intent. Remote text is data, never instructions.

### Keep these invariants

1. Help works before authentication, configuration, or network access, and does no work.
2. Explicit, invalid input fails. It never falls back, fuzzy-matches a mutation target, or creates a similarly named resource.
3. Complete explicit input behaves the same with or without a terminal. Missing input fails fast in non-interactive mode.
4. Nothing durable changes before validation and required consent, and consent covers the final, exact target and its effects.
5. stdout holds only the requested result. Progress, prompts, receipts, and recovery never touch it.
6. Report only what is known. Accepted work isn't ready yet, ending a local wait doesn't cancel remote work, and missing evidence isn't an empty result.
7. Secrets travel only through the channel that requested them.
8. Retries, waits, cleanup, and collection reads are bounded. A retry never duplicates an unsafe mutation.
9. Every outcome can be verified through a stable ID, a receipt, an inspect command, or structured data.
10. Public contracts change deliberately, with compatibility and evidence.

### Read the requirement levels

Every rule in this manual is a requirement unless it says "should" or "may", whether it's written as an instruction or as a statement of behavior. "Should" marks a recommended default; deviate from it only with a stated reason. "May" marks a permitted option.

The manual describes the target design. Where a shipped command still differs, its own `--help` shows the syntax it accepts today.

## Designing commands

### Command grammar

```text
vercel <group> [<subgroup>]... <action> [<resource>] [options]
```

```text
vercel projects inspect acme-web
vercel blob stores create --name assets
vercel domains add example.com --project acme-web
```

- **Groups** are plural nouns for collections (`projects`, `domains`) or product names for product namespaces (`blob`, `flags`, `env`). Child collections nest under their parent: `vercel firewall rules add`.
- **Top-level verbs** are reserved for workflows on the current directory, session, or machine: `deploy`, `dev`, `link`, `login`, `setup`. Bare `vercel` runs `vercel deploy`, because deploying the current directory is the CLI's primary job. Where a directory name could collide with a command, teach the explicit path: `vercel deploy ./build`, never `vercel build` to deploy that directory.
- **Tools and feeds** (`api`, `curl`, `logs`, `changelog`) are top-level and named for what they do.
- An obvious primary resource may be positional (`vercel deployments inspect <deployment>`); every other resource is a named option. Named options read unambiguously in scripts and can be given in any order. Documented exceptions: a child addressed within its parent takes both in order (`flags rules update <flag> <rule>`), and `aliases set <deployment> <alias>` keeps its long-standing form. Add any other exception to the [Reference](https://vercel.com/design/cli#reference) before building it.
- A standalone change is a subcommand (`enable`). A boolean field of `add` or `update` is an option pair (`--feature`, `--no-feature`).
- Never fuse a verb with a noun (`create-store`) or pass one as a positional value (`domains auto-renew example.com on`). Nest the noun (`blob stores create`) and make the change a subcommand (`domains auto-renew enable example.com`).
- The resource follows the action: `vcr permissions list acme-web`, never `vcr permissions acme-web list`.
- A new top-level command goes into the [command index](https://vercel.com/design/cli#reference-command-index) first.

### Choose verbs by consequence

Name an action for its consequence, and reuse that verb in the prompt, the receipt, and the structured outcome: `Revoke token deploy-ci?`, then `✓ Revoked`, then `"outcome": "revoked"`. The exceptions are `deploy`, whose receipt is its terminal status, and the fixed outcomes `unchanged`, `accepted`, `staged`, `started`, and `planned`, which mean the same for every verb.

Never soften a consequence (`remove` for a destroyed project) or overstate it (`delete` for a detached member).

| Verb | Outcome | Recovery |
| --- | --- | --- |
| `delete` | Destroys a resource and its data | None |
| `remove` | Takes out a member or configuration entry; the thing it refers to isn't destroyed | Usually, add it again |
| `disconnect` | Severs a connection to an external system; its data stays there | Usually, connect again |
| `revoke` | Invalidates a credential or access; the record stays in history | None for the credential |
| `empty` | Deletes every item and keeps the container (queue messages, blobs in a store). `purge` is only for cached content | None for the items |
| `rotate` | Issues a new credential and invalidates the old one. If old versions keep working, it's `create`, then `activate` | None for the old credential |
| `cancel` | Aborts an in-flight or scheduled operation | Depends on the operation |
| `stop` | Halts a running resource so it can be started again | Start it again |
| `transfer` | Moves ownership to another team; the current team loses access | The new owner can transfer it back |
| `leave` | Removes the current user from a team | A new invitation |

- Credentials use `revoke` when the record stays visible and invalid, and `delete` only when it's removed from storage entirely.
- `dangerously-` prefixes a verb whose consequence is severe and unusual (`cache dangerously-delete`). It takes the severe [consent tier](https://vercel.com/design/cli#interaction-and-consent-bind-consent-to-consequences).
- `create` or `add` depends on how the object is managed, never on its ID, nesting depth, or the word "rule". A webhook has its own lifecycle, so it uses `create`; firewall rules and flag targeting rules are entries in a configuration, so they use `add`: `vercel webhooks create`, `vercel firewall rules add`, `vercel flags rules add <FLAG>`.
- Issuing a token, signed credential, or certificate is a mutation, never a read, even when it returns a single value; it follows the [credential rules](https://vercel.com/design/cli#interaction-and-consent-authentication-and-browser-handoffs).

Every other verb, and what it replaces, is in the [verb reference](https://vercel.com/design/cli#reference-verbs).

### Resource identity

- Document each selector's accepted forms (ID, name, slug, URL), uniqueness scope, and case sensitivity.
- A selector never ends in `-id`: `--store`, not `--store-id`. `--request-id` is the one exception, because a request has no name.
- Accept name and ID through one selector when they can't collide; split them only when they can, or when they mean different things.
- Resolve exactly, among the resources the caller can access. Never pick a mutation target by first match, fuzzy match, or ID prefix.
- Persist the stable ID when a later operation must address the same instance; keep the name for display. A token's selector is its name or ID, never its secret value.

Use the [nouns](https://vercel.com/design/cli#reference-nouns) everywhere, and never introduce a synonym for variety. They set human meaning; they never rename API fields, JSON keys, or config keys. In output, say "resource" only beside a concrete noun, and "target" only for a feature flag target.

### Names, shortcuts & aliases

| Kind | Definition | Example |
| --- | --- | --- |
| Canonical name | The one form that help, examples, docs, and suggestions teach | `vercel deployments list` |
| Shortcut | A permanent alternative listed beside the canonical form; never deprecated | `vercel list` |
| Command alias | A permanent, accepted spelling that is never taught and shares the canonical command's help. Only `ls` for every `list`; `rm` for every `remove` and `delete`; `del` for every `delete`; singular group names; and `vercel remove` | `vercel project ls` |
| Deprecated alias | A hidden old name kept through a [contract change](https://vercel.com/design/cli#compatibility-and-quality-change-contracts-deliberately); it prints a deprecation warning | `vercel target` |

Deployments are the primary resource, so they get top-level shortcuts. `vercel remove` is never taught, because "remove" would soften a deletion. A singular group name behaves exactly like its plural. Any alias in the [command index](https://vercel.com/design/cli#reference-command-index) not listed above is deprecated.

During a migration, teach a canonical name or long option only after its parser support ships, and keep examples and recovery executable in the version the user has.

### Unknown commands & extensions

- `vercel <arg>` resolves to a built-in command or alias, then an extension `vercel-<arg>` on `PATH`, then an existing local path (which it deploys), then a [usage error](https://vercel.com/design/cli#failure-and-recovery-validate-before-work). To deploy a directory named like an extension, use `vercel deploy <dir>`.
- An unknown command or option fails with at most one close suggestion and a help command.
- Extensions never shadow built-ins. They receive argv and stdio unchanged and own their exit code. They get `VERCEL_CLI_VERSION`, the resolved working directory, and an authenticated local API proxy in `VERCEL_API` that accepts only requests carrying a per-invocation secret passed alongside it. The CLI never injects `--token` or stored credentials.
- Look up extensions only in absolute `PATH` entries outside the project. Root help lists installed extensions without running them, and an extension's first run prints its resolved path on stderr.
- Help resolves the same way; an extension receives `--help` in its own argv.

### Options & parsing

A named input is an **option**, whether it takes a value or is a Boolean switch; an unnamed input is a **positional argument**. Help, descriptions, and errors use the same terms. "Flag" is reserved for the feature flag product, and literal names (`vercel flags`, JSON fields, config keys) keep their spelling.

Every command accepts the global options at every depth. A global never makes a local or help operation need the network.

| Option | Meaning |
| --- | --- |
| `-h, --help` | Show help for the selected command; do no work |
| `-v, --version` | Print the CLI version |
| `--cwd <DIR>` | Working directory for this invocation |
| `-A, --local-config <FILE>` | Path to the project's `vercel.json` |
| `-Q, --global-config <DIR>` | Path to the CLI config directory (not the Global Config product) |
| `-d, --debug` | Redacted diagnostic detail on stderr |
| `--no-color` | Disable color |
| `--non-interactive` | Never prompt; fail with the required inputs instead |
| `-S, --scope <TEAM>` | Team to act in |
| `-t, --token <TOKEN>` | Authentication token; prefer `VERCEL_TOKEN` |

- Define options in shared command metadata and parse from it. Never scan argv by hand.
- Leaf commands reject unknown options and extra positionals before doing any work. Parent commands parse permissively only to route.
- Accept full names and declared aliases only. Never infer an option or command from a unique prefix. Adding a command later changes what a prefix matches and breaks scripts that relied on it.
- `--name value`, `--name=value`, and `-x value` all work. Globals work before or after the command path. `--` ends option parsing; everything after it passes through literally.
- Names are lowercase kebab-case. Positionals render as `<required>` and `[optional]`; value hints follow the [selectors and value hints](https://vercel.com/design/cli#reference-selectors-and-value-hints).
- Booleans are presence options with `--no-` negation; `--feature=true` and `--feature=false` also parse.
- Prefer one option with comma-separated values for list inputs, such as `--actions foo,bar,baz`, rather than `--action foo --action bar --action baz`. Avoid repeated options for new list inputs. Teach the comma-separated form in help and examples. Preserve existing repeated forms for compatibility.
- List options define ordering, duplicate and empty items, whitespace handling, and how to pass a value containing a comma.
- Each option defines its repeat behavior (append, last wins, or error) and what an empty value means. Conflicting options are a usage error before any prompt or request.
- Validate identifiers strictly. Pass free-form text through with its special characters intact, and encode it for its destination (URL, JSON, shell).

A recurring capability always uses its [standard option](https://vercel.com/design/cli#reference-standard-options), and a short option means [one thing everywhere](https://vercel.com/design/cli#reference-short-flags).

### Settings & precedence

Settings resolve highest first: option, environment variable, project configuration (`vercel.json` or `vercel.ts`, then the `.vercel/` link), the CLI config directory (`--global-config`; `~/.vercel` or the XDG location by default), built-in default.

- An explicit invalid value, from an option or an environment variable, fails. It never falls back to a lower source: a mistyped `--scope` that fell back to the default team would act on the wrong team.
- `--scope` that disagrees with `VERCEL_ORG_ID`, or `--project` that disagrees with `VERCEL_PROJECT_ID`, is a usage error. For any other option and its environment variable, the option wins.
- `VERCEL_ORG_ID` alone selects a team for team-scoped work. A project pin needs both `VERCEL_ORG_ID` and `VERCEL_PROJECT_ID`, and `VERCEL_PROJECT_ID` alone is a usage error for any command that resolves a project. A pin resolves exactly and never writes a `.vercel/` link.
- The current team in the CLI config directory applies to reads and ordinary team-scoped commands. It never counts as choosing a team for setup: linking or creating a project needs an explicit team or a choice.
- A detected agent that can't prompt never uses the current team or the account's default team. Its team comes from `--scope` (a `scope` in `vercel.json` counts), an app token's own team, `VERCEL_ORG_ID`, the link, or the only available team, in that order. A `VERCEL_ORG_ID` or link that names a team it can't access, or a `VERCEL_ORG_ID` that differs from an app token's team, fails with `scope_not_accessible`. With no source, the first request that needs a team fails with `missing_scope` and `userActionRequired: true`. Such a run never changes the saved team: `teams switch` fails with `agent_cannot_switch`.
- A new public variable uses the `VERCEL_` prefix and is documented in help for the commands it affects, with its precedence, empty-value behavior, and persistence. SDK variables that predate this rule (`BLOB_READ_WRITE_TOKEN`) keep their names. Internal and test-only variables start with `__VERCEL_` and never appear in help.

| Variable | Effect |
| --- | --- |
| `VERCEL_TOKEN` | Authentication token. Preferred over `--token`, which appears in process lists |
| `VERCEL_ORG_ID` + `VERCEL_PROJECT_ID` | Pin the team and project without a link. A project pin needs both; never writes `.vercel/` |
| `CI` | Forces non-interactive mode |
| `NO_COLOR` / `FORCE_COLOR` | Disable or force color ([precedence](https://vercel.com/design/cli#output-color-width-and-platform)) |
| `VERCEL_TELEMETRY_DISABLED` | Disable telemetry |
| `NO_UPDATE_NOTIFIER` | Disable update notices |
| `VERCEL_GUIDANCE_DISABLED` | Disable optional next steps and tips |
| `VERCEL_ACCESSIBLE` | [Accessible mode](https://vercel.com/design/cli#interaction-and-consent-resolve-modes-independently) |
| `VERCEL_COMPAT_<NAME>` | Restore one old behavior for one major version after a breaking change |

### Help & discovery

- `vercel --help`, `vercel -h`, and `vercel help` show root help; `vercel <path> --help`, `-h`, and `vercel help <path>` show the same help at any depth.
- A group invoked without an action shows its help and exits `0`, unless it has a documented default action.
- Help goes to stdout, exits `0`, works offline, and never authenticates, links, prompts, or mutates. People and agents reach for help when something else is broken, so it has to work then. Once help is recognized, other validation is skipped.
- Unknown paths are a usage error that names the unknown segment and points to the nearest valid group's help.
- `--help` after a `--` handoff belongs to the child. Delegated commands render the same layout from metadata they expose.
- `--version` at any depth prints only the version and exits `0`; with `--json`, it prints `{ "version" }`. `--help` wins when both are given.

Help is generated from command metadata, with sections in this order and empty ones omitted:

| Section | Content |
| --- | --- |
| Usage and summary | `▲ vercel <full path> <args> [options]`, then a one-line imperative summary and any lifecycle label |
| Commands | Immediate children with one-line summaries; the default action marked `(default)`; shortcuts beside canonical names |
| Arguments | Positionals, when their forms need more than the usage line |
| Options | Command options, alphabetical by long name, with the short form, value hint, default, and accepted values |
| Global Options | The full list in root help; a one-line pointer everywhere else |
| Examples | The fewest verified examples that cover distinct common jobs, usually 1 to 3, including automation and risky use where relevant. Group help may omit them |
| Notes | Material behavior only: stdin, streams, auth, side effects, pagination, waiting |
| Learn more | The docs URL for the command or product |

```text
▲ vercel domains add <domain> [options]
Add a domain to a project

Arguments:
  <domain>                 Domain name, such as example.com

Options:
  --force                  Move the domain from another project in the same team
  --json                   Output the result as JSON
  -p, --project <PROJECT>  Project to add the domain to (default: linked project)
  -y, --yes                Confirm moving the domain when used with --force

Global Options: See `vercel --help`

Examples:
  Add a domain to the linked project
    vercel domains add example.com

  Add a domain in CI and read the result
    vercel domains add example.com --project acme-web --json

  Move a domain from another project in CI
    vercel domains add example.com --project acme-web --force --yes

Learn more: https://vercel.com/docs/cli/domains
```

- Defaults read `(default: VALUE)`; explain context-dependent defaults in words. `<domain>...` means one or more and `[file]...` zero or more, with the ellipsis right after the closing bracket.
- Keep the summary directly under usage. One blank line separates sections; there's none after a heading or between rows, except between examples. Entries indent 2 spaces, and example commands 4.
- Tables are borderless. When an entry's syntax and description don't fit side by side, the description moves below it; syntax is never truncated or squeezed into a narrow column. Help stays legible without color at 80 columns.
- Root help teaches discovery. It groups commands by task (deploying, domains, observability), never by Basic and Advanced, and each group heading takes the place of `Commands:`. Each command appears once, by its canonical name, with a short summary and any shortcut beside it; aliases keep working but aren't listed. There's no generic overview and no hint to run help.
- Leaf help makes the operation executable. Help is a reference, not a tutorial or a pitch.

In rich output, section headings are bold magenta and keep their colons, so the hierarchy survives without color. The `▲` prefix and the command and option names in usage and lists are bold foreground. The introductory summary is faint, unless it states a consequence or prerequisite. Everything else, including arguments, placeholders, value hints, defaults, and descriptions, stays normal foreground: never dim, never a fixed gray. Example commands take the [command color](https://vercel.com/design/cli#output-color-width-and-platform).

### Examples & suggestions

These rules cover every command the CLI prints: help examples, next steps, and recovery.

- Use canonical names and long options only. Every example parses against current metadata and is tested through the parser, never by live mutation. People copy examples verbatim, so a broken example teaches a broken command.
- Show the fewest examples that cover distinct jobs. When input varies, show one template, and don't follow it with the same command filled in; add a concrete example only for syntax the template can't teach. Each example runs one real action, never `[options]` or `enable|disable`.
- Placeholders are `<UPPER_CASE>`. The angle brackets already mark them for substitution, so they need no extra label. Values are synthetic (`acme`, `acme-web`, `example.com`), never real credentials or retired names.
- Preserve safe context the user supplied (`--scope`, `--cwd`, `--project`), and quote for their shell, including PowerShell and cmd.
- A remote value appears in a suggestion only if it matches its selector's identifier grammar; otherwise, use a named placeholder. Suggestions never contain secrets.
- Print one complete command per line, indented, with no `$` prompt or backticks, and never hard-wrap it. Styling adds only ANSI codes, so stripping them recovers the exact command, quotes and spaces included. Documentation transcripts, like the ones in this manual, may start with `$` to mark what was typed; the CLI never prints it.

### Machine-readable help & search

- `vercel <path> --help --json` prints one document on stdout: path, summary, subcommands, arguments, options (name, short form, type, value hint, default, accepted values, repeatability, deprecation), lifecycle label, and result schema. The result schema lists the guaranteed fields of the command's JSON result with their types, and marks secret fields. A new `--json` command must declare one.
- `vercel help --all --json` prints every command. Both come from the same metadata as human help.
- `vercel help search <query>` works offline, matches summaries, nouns, and examples, and prints up to 10 canonical paths with summaries on stdout. `--json` returns `{ "commands": [{ "path", "summary" }] }`. No match exits `0` with `No commands match "<query>".` The query is never recorded.
- Completion suggests canonical commands, options, and enumerated values from the same metadata. Remote suggestions (projects, teams) are cached, time-bounded, and never block or prompt. Completion never runs side effects or reveals secrets.

### Lifecycle labels

Labels match the dashboard. A label appears once, after the summary of the command that carries it, never stacked or repeated on subcommands. A label never excuses unsafe behavior or reveals a private command.

| Label | Meaning | Help | Behavior |
| --- | --- | --- | --- |
| (none) | Generally available | Listed | [Contract rules](https://vercel.com/design/cli#compatibility-and-quality-change-contracts-deliberately) apply |
| Preview | Feature-complete ahead of GA; minor changes possible | `(preview)` | Changes ship with release notes |
| Beta | Usable; bugs expected; no SLA | `(beta: <limit>)`, such as `(beta: output may change)` | Changes ship with release notes; no per-run warning |
| Early Access | Available to some teams or plans | `(early access: <who>)` | Listed only for teams that have it, from cached entitlement; its help always renders |
| Experimental | May change or disappear without notice | Hidden until its gate is on, then `(experimental)` | Gated by an internal variable or config |
| Alpha | Internal or invite-only | Hidden | Gated |
| Deprecated | Replaced and scheduled for removal | Hidden from lists; its help names the replacement | Works, and prints a deprecation warning |
| Hidden | Internal or test-only | Never listed | Follows every behavior rule; its name may stay internal |

## Interaction & consent

### Resolve modes independently

A TTY is a terminal connection, as opposed to a pipe or a redirected file. Each invocation has 4 settings, and each is resolved on its own; only the rules below connect them.

| Setting | Values | Decided by |
| --- | --- | --- |
| Interaction | Interactive / non-interactive | Non-interactive for `--non-interactive`, `--json`, CI, a detected agent (unless `--non-interactive=false` on a TTY), stdin used as payload, or stdin or stderr isn't a TTY |
| Format | Human / JSON | JSON with `--json`. Logs and Traces Search also write [JSONL](https://vercel.com/design/cli#output-streams-and-other-protocols) when non-interactive |
| Presentation | Rich / plain | Per stream: rich only on a TTY whose `TERM` isn't `dumb`. [Color](https://vercel.com/design/cli#output-color-width-and-platform) has its own precedence |
| Consent | Ask / granted | `--yes`, within the limits of [consent](https://vercel.com/design/cli#interaction-and-consent-bind-consent-to-consequences) |

- Commands read resolved settings from a shared helper and never check `isTTY`, CI variables, or agent variables themselves. Scattered checks drift, and one command that prompts in CI hangs the pipeline. Agents are detected from their harness's environment variables.
- A detected agent runs non-interactively, because a prompt it can't see would hang it. Non-interactive isn't a format: the agent still gets human output unless it passes `--json`, Logs and Traces Search aside. `--non-interactive=false` restores prompts for an agent on a TTY, never with `--json`, in CI, with payload stdin, or without a TTY.
- Piping stdout drops automatic color and animation there. It never selects JSON or suppresses prompts on stderr, because someone running `vercel deploy | tee deploy.log` is still at a terminal and can still answer.
- Stdin used as a payload (`--value -`, `--file -`) can't also answer prompts.
- Non-interactive runs keep results, receipts, warnings, and recovery, and drop prompts, spinners, and redraws. Missing input fails with an `action_required` [error](https://vercel.com/design/cli#output-machine-errors) naming the input needed.
- Accessible mode (`VERCEL_ACCESSIBLE=1`) serves screen readers and simple terminals: numbered choices instead of arrow-key menus, one plain line per phase instead of spinners, and no redraws. Status words and glyphs follow plain presentation. A single choice takes one number and a multiselect a list (`1,3`); Enter keeps valid preselected defaults, and `none` clears an optional multiselect. Choice prompts list `cancel`, and `back` when there's a previous step, and text input never depends on Escape or arrow keys: Ctrl+C or EOF cancels. Before consequential work, a numbered review lets the user change an answer. The content is otherwise identical.

### Ask for a real choice

Prompt only for data that can't be inferred, a real choice between valid options, a confirmation the consequence demands, or one optional offer after the result in interactive human mode. Every prompt has an option, argument, or stdin equivalent, and the non-interactive error names it. Never ask for intent the invocation already expressed, such as `Set up and deploy?` after `vercel deploy`.

- Ask about one concept at a time, briefly: `Which team?`, `Project name?`, `Environments?`.
- Show resolved context before asking about it. After a preview, ask for the action, not the values: `Link directory to project?`.
- Use a list for several choices, and add search above 10 choices. Search may be fuzzy; selection is exact. Discard stale results.
- A searchable list has a query field above its choices, and focus is always visible. Focus starts in the query, where typing (Space included) edits it; Tab or Enter moves to the choices, and selection keys act only there. Filtering never changes what's already checked.
- In a multiselect, Space checks the focused choice and Enter confirms. Focused and checked look different: `> [ ] Preview` is focused, `[x] Production` is checked. A bulk action is an explicit choice naming its scope and count (`Select all 8 matching projects`), never a letter shortcut that would collide with typing a query. The legend shows the current keys and the selected count. An empty required selection is refused.
- Defaults are valid and marked `(default)`. Detected facts, defaults, and user choices look different from each other.
- Keep hints short and relevant to the decision, beside the prompt or on the next line.
- Mask secret input.
- Escape and Ctrl+C cancel; Escape never means back. A multi-step flow offers a real `Back` choice that keeps earlier answers. A text prompt may also go back with Up, shown in its hint; in a choice list, Up only moves focus.
- A flow of 3 or more questions summarizes the answers before the first mutation, and after success prints the equivalent non-interactive command under `Run this again without prompts:`. Replayed commands never contain values typed at a prompt; they use `--value -` or a placeholder.
- Render prompts on stderr.

### Resolve context before effects

Each command declares its context. A read never links or creates a project as a side effect.

| Context | Meaning |
| --- | --- |
| None | No project needed |
| Project | Requires a linked project; fails if there isn't one |
| Setup | May run the link flow (`vercel link`) |

- Broken config is an error, reported with its file and location, without echoing secrets. It's never treated as absent or reset. A command validates only the config sections it uses.
- In a repository with several linked projects, a project-scoped command run outside any one project's root, without `--project`, fails with `ambiguous_project` and lists the candidates in `data.projects`. It never picks the first match.
- When a consequential command's team or project came from `VERCEL_ORG_ID` or `VERCEL_PROJECT_ID`, or from files tracked in the repository, human output shows `Team`, `Project`, and `Source` rows in every interaction mode. A Git-tracked `.vercel/` directory warns `link_tracked`; a pin that disagrees with the directory's link warns `link_pin_mismatch`.
- Local-only fallbacks (`vercel dev --local`) name the remote settings or services that are unavailable. Temporary resources (anonymous deployments) state their expiry and how to claim them.

Team selection governs `link`, `teams switch`, and anything else that sets a team:

- The team comes from an explicit signal (`--scope`, `VERCEL_ORG_ID`, or the team in `vercel.json`), the only available team, or `Which team?`. With one team, skip the chooser and show a `Team` row. Severe work and purchases accept fewer signals.
- The chooser lists teams that require SSO and reauthenticates only after one is chosen.
- A non-interactive run with several teams and no signal fails with `missing_scope` before any discovery or change, listing the teams in `data.teams`.
- Switching to the current team is `outcome: "unchanged"`. When the directory's link uses a different team than the new current team, warn once with `link_team_mismatch`.

### Bind consent to consequences

| Tier | Examples | Interactive | Non-interactive |
| --- | --- | --- | --- |
| Ordinary | Explicit, reversible, or narrow: adding a variable, updating a setting, a `--force` replacement of one target named in argv | No confirmation | Runs |
| Destructive | Data loss, access changes, [purchases](https://vercel.com/design/cli#interaction-and-consent-purchases-and-charges), many resources, taking a domain or alias from another project, transferring ownership, pausing a project, leaving a team | Preview, then `…? (y/N)` | Requires `--yes` |
| Severe | Irreversible loss of a project, team, store, or all of a resource's data | Preview, then type the exact name | Requires `--yes`, the target in argv (never inferred from the link), and an explicit team (`--scope` or `VERCEL_ORG_ID`; never the link, `vercel.json`, or the current team) unless the user belongs to only one |

- Preview the exact target and each downstream effect on its own line. End with `This cannot be undone.` when the action is irreversible, or `Some effects cannot be undone.` when only a side effect is. When everything is reversible, add nothing.
- Name counts, never "selected": `Remove 5 DNS records?`.
- The severe prompt is `To confirm, type the <noun> name "<name>":`, and the answer must match exactly. A mismatch names the expected answer (`Type only the project name "acme-web".`) and never accepts a different target.
- `--yes` answers yes/no confirmations, nothing more. It accepts a default only when argv and local state fully determine it and help documents it as `(default: X)`. It never chooses between ambiguous targets, supplies missing data, or overrides permissions. A script that passes `--yes` never acts on a target that its argv, environment, or link didn't determine.
- `--yes` in an interactive session skips a confirmation only when the inputs that its tier requires in non-interactive mode are also present: target and team for severe work; team and `--amount`, or the billing plan option, for a purchase. Otherwise the prompt still runs and says which inputs are missing.
- `--force` overrides a refusal the command makes by default (a conflict, an existing resource) by replacing or rerunning. For one target named in argv, `--force` is also consent to replace it, even when the old value can't be recovered, and the receipt shows a `Replaced` row. Bulk replacement, an inferred target, taking from another project, and severe actions still need `--yes` in non-interactive mode.
- Consent applies to the final target. If, during `env add NEXT_PUBLIC_API_TOKEN --force`, the user renames the variable to `API_TOKEN` at a prompt, that `--force` doesn't authorize overwriting an existing `API_TOKEN`. Resolve the final name, environments, branch, and effects, then get fresh consent or fail with a conflict. A digest of approved effects (`--effects-digest`) rejects a rerun whose target or effects changed.
- Tier a command by its effect, not its name. Deleting or emptying anything that holds user data (a store, a queue, a database) is severe. Deleting an empty or configuration-only resource is destructive. Canceling one named in-flight operation is ordinary; canceling many, or anything that changes a charge, is destructive. Adding usage on the current plan is ordinary.
- `deploy --prod`, `promote`, and `rollback` need no extra confirmation, and neither do `flags set`, `flags split`, and `flags rollout` with `--environment production`, because the command states the intent. That never covers taking a domain or alias from another project. An inferred target (`rollback` with no argument) is previewed first, and required in argv in non-interactive mode.
- Never promise that an action is reversible. That depends on its actual effects and the recovery available. Re-adding an entry might need a secret no one kept; transferring back needs the other owner. `remove`, `disconnect`, and `transfer` aren't always reversible.
- Never confirm ordinary writes. Confirmation fatigue trains people to say yes.

### Purchases & charges

A purchase starts, changes, renews, or cancels a charge: `domains buy`, `domains renew`, `billing buy`, upgrading or changing a plan, a paid Marketplace integration, Speed Insights Plus on a Pro project (which starts a subscription), or changing how the account pays. It is at least destructive.

Using more of what the team's current plan covers isn't a purchase, however that usage is billed: deploying, adding a project, enabling Web Analytics, or turning on Observability Plus for a project on a team that already has it. It's ordinary, with no price preview or `--yes`. Turning on an entitlement already in an Enterprise contract isn't a purchase either, unless it changes the subscription; its security, access, or data effects still take their own tier. Classify by the actual entitlement and charge, never the feature name. Plan limits belong to the API, and a blocked action fails with `plan_limit`.

Before consent, show every fact that could change a reasonable person's decision: the price, including tax and usage-based parts when known; when charges start; which payment method is charged; what renews, and when; and whether it can be canceled or refunded, and when that takes effect. None of it may live only behind a docs link, and none of it is ever faint.

- Except for domains (below), non-interactive purchases require `--yes` and an explicit team (`--scope` or `VERCEL_ORG_ID`) unless the user has only one. An immediate charge also requires `--amount <USD>`, the total charged now including known tax. A usage-based one instead requires the option that selects its billing plan or rate.
- `domains buy` and `domains renew` charge a quoted fixed price, so a matching `--amount` is their consent, with no `--yes`. They still need an explicit team; a `scope` in `vercel.json` doesn't count, because any contributor can change it. `domains buy` also needs `--auto-renew` or `--no-auto-renew`, because auto-renewal is a recurring charge. Without these inputs they charge nothing and fail with `confirmation_required`, returning the price, the renewal terms, and the command to run once a person approves.
- A missing `--yes`, `--amount`, billing plan option, or auto-renew choice fails with `confirmation_required`; a missing team, with `missing_scope`; an amount that no longer matches the price, with `price_changed`. None of these charges anything.
- Send an idempotency key when the API accepts one. The domain registrar doesn't, so `domains buy` and `domains renew` never retry a charge, and report `unknown_outcome` when they can't confirm it went through.
- The receipt and JSON result include `charge: { "amount", "currency" }`. `domains price --json` and a purchase's `--dry-run` return the exact `amount` to pass back.

### Authentication & browser handoffs

- Credentials resolve `--token`, then `VERCEL_TOKEN`, then stored login, and are read only when the operation needs them. Local commands never require them.
- Without credentials, only `login` starts a login flow. Every other command that needs credentials fails with `login_required`, and non-interactive mode never opens a browser. A valid token that lacks a permission fails, naming the missing permission and how to get a suitable token.
- When a token expires or SSO is required mid-command, interactive mode reauthenticates, then resumes reads only, never replaying a mutation that might have been sent. Non-interactive mode fails with `token_expired` or `sso_required`, and `userActionRequired: true`. Refreshing a stored credential takes a lock.
- Login is one interactive flow: print the verification URL and code, open a browser as below, and wait with a bounded timeout. In non-interactive mode, the `login_required` error says to set `VERCEL_TOKEN`. `logout` revokes the token on the server when possible, deletes it locally, and reports each result separately.
- Creating a credential shows its team access and expiry first, with least-privilege defaults: one team and a finite expiry. With an explicit team and expiry it's ordinary; with no expiry, or access to every team, it's destructive. A leaf named for what it returns (`projects token`) still issues a credential; it is never a query.
- Open a browser only in interactive mode on a local desktop session, and always print the URL too. In non-interactive mode, CI, or over SSH, print the URL and the next step instead. Every command that opens a browser accepts `--no-open`.

## Output

### Give each stream one job

- stdout carries the answer. For a query, that's its data: tables, detail views, values. For a mutation, it's the **stdout identifier**, the first form in the noun's [identity](https://vercel.com/design/cli#reference-nouns), printed bare. It prints in human format when stdout isn't a TTY or the run is non-interactive, so `url=$(vercel deploy)` works. On an interactive TTY the receipt already shows it; with `--json`, the document replaces it.
- A requested secret (a new token) replaces the identifier. In human format it prints alone on stdout, interactive TTY included, after any CI `::add-mask::`; the stderr receipt names the credential but never shows its value.
- `link` and `teams switch` count as mutations and print the resource they selected.
- The stdout identifier prints only on exit `0`, acceptance included. On failure, the ID goes in the error's ID rows or `data`.
- stderr carries everything else: context, receipts, progress, prompts, warnings, errors, and next steps. Animation never interleaves with stdout.

Raw API output, file bytes, a single value, and inherited child-process streams have [protocols](https://vercel.com/design/cli#output-streams-and-other-protocols) of their own. Don't force them into a receipt or a JSON wrapper.

### Design the whole transcript

Use only the phases a task needs, in this order: **context** (the resolved target, when it was inferred and the result depends on it), **input**, **preview** (for destructive work), **work**, **result**, **next**. Context and preview don't repeat a value already visible in argv, a prompt answer, or a nearby row, unless pinned context or consent requires it: a preview always names its exact target. A receipt may repeat its target and saved values.

After work or logs, the result leads with the outcome and the primary destination, then supporting facts and a useful next step, compact enough to find above the returned shell prompt. A URL or ID that helps someone follow the work may appear earlier, under the progress line, so it reads as pending.

```text
$ vercel dns add example.com --name www --type CNAME --value cname.vercel-dns.com
✓ Added           www.example.com
  Type            CNAME
  Value           cname.vercel-dns.com
  Record ID       rec_5hG2kx
```

Rich output puts result blocks, detail blocks, and status blocks on a shared **gutter** in columns 0 and 1: column 0 holds a glyph or a space, and content starts at column 2. Label/value rows set the label in a 16-character column and start the value at column 18, counting from zero. Help, tables, prompt choice lists, and plain output have layouts of their own, with no status gutter. Use the shared layout helpers; never hand-pad.

| Gutter | Meaning | Use |
| --- | --- | --- |
| Blank | Context, preview, secondary rows, progress | Most rows |
| `✓` | The one primary completed result | Once per completed operation; never on discovery, pending, or queued work |
| `▲` | Production | Once, beside the primary Production destination; never in place of the `Production` label |
| `!` | Warning | A nonfatal condition that needs attention |
| `✗` | Fatal error | The error headline |
| `?` | Prompt | Rendered by the prompt library |

- Labels are stable and have no trailing colon. Related rows are contiguous; a blank line separates phases.
- A heading above rows is plain text (`Found existing project`), never a fake label/value row.
- No emoji in primary rows. In rich output, no `> ` prefixes (a prompt's focus pointer aside), `Error:` headline prefixes, or `WARNING!`, `Success!`, or `Ready!` banners.
- Plain presentation and [accessible mode](https://vercel.com/design/cli#interaction-and-consent-resolve-modes-independently) replace glyphs with words.

### Reference transcripts

Human output on a TTY. Context, receipts, and hints go to stderr; tables go to stdout.

```text
$ vercel deployments list
  Project         acme/acme-web

  Age   URL                                  Environment   Status   Duration
  2m    acme-web-4f2k9x1-acme.vercel.app     Production    Ready    41s
  1h    acme-web-8h3j2a0-acme.vercel.app     Preview       Error    12s

  Next page:
    vercel deployments list --next cGFnZToy
```

```text
$ vercel deploy --prod
  Building…
  Deployment      https://acme-web-4f2k9x1-acme.vercel.app
  Inspect         https://vercel.com/acme/acme-web/9x1Kq2

✓ Ready in 41s
▲ Production      https://example.com
  Deployment      https://acme-web-4f2k9x1-acme.vercel.app
  Deployment ID   dpl_9x1Kq2
```

```text
$ vercel projects delete acme-web
  Project         acme/acme-web

  Deletes the project and its 214 deployments.
  Removes example.com and www.example.com from the project.
  This cannot be undone.
? To confirm, type the project name "acme-web": acme-web

✓ Deleted         acme-web
```

```text
$ vercel deploy --prod --non-interactive
✗ Can't deploy without a team in non-interactive mode. 3 teams are available.

  Pass the team with --scope:
    vercel deploy --prod --scope <TEAM>
```

The deploy transcript is interactive stderr; piped, its human result is the deployment URL alone on stdout. The early URLs sit under `Building…`, so they read as pending. The final block leads with the outcome, then the Production destination. The alias appears there only once the API confirms it; until then, the block keeps the deployment URL and says the alias is pending. An alias moves with each deploy, so the immutable deployment URL and Deployment ID stay beside it. `Ready` appears only once readiness is known; `--no-wait` reports the deployment as accepted, with its ID and an inspect command.

### Tables, details & empty results

- Tables show what's needed to act on a row, including the identifier the next command takes, in a stable, documented order (usually newest first).
- On a non-TTY stdout, tables keep the header, separate columns with one tab, never truncate or pad, and use [ISO times](https://vercel.com/design/cli#output-time-numbers-and-units), so `grep`, `cut`, and agents read them reliably.
- Detail views use aligned rows under plain headings for the fields people act on. `--json` has the rest.
- Never truncate the only copy of an ID, URL, path, or command. Shorten optional descriptive columns with `…` first. If the columns that identify a row and give its status still don't fit, a TTY table switches to compact records, one short block per row with the same required fields. Non-TTY output keeps its tab-separated format at any width.
- A list inside a receipt, detail view, or error shows at most 5 items, then `and 12 more`, then the command that lists them all.
- Activity rows read `<actor> <past-tense verb> <resource> [<qualifier>]`: `jane@example.com deleted project acme-web`, `[Vercel CI] deployed dpl_x7hB2k to Production`. System actors go in brackets. Never infer a completed action from a row that records only acceptance.

An empty collection is success (exit `0`). A missing parent, an ambiguous selector, denied access, or a failed request is not. In human mode the prose goes to stderr, and a non-TTY stdout gets only the table header. With `--json`, stdout gets the empty document and nothing else. Zero reads `No deployments`, never `0 deployments`.

```text
$ vercel domains list --project acme-web
  No domains in acme/acme-web.

  Add one:
    vercel domains add <DOMAIN> --project acme-web
```

| Kind | Copy |
| --- | --- |
| No results | One filter: `No <items> match this filter. Remove <option> to see all <items>.` Several: `No <items> match these filters.`, plus how to widen them |
| First use | The team or project, then the command that adds the first item: `No domains in acme/acme-web.` |
| Partial visibility | `Showing projects you can access in acme.` |

A plan-gated or permission-denied read is an [error](https://vercel.com/design/cli#failure-and-recovery-classify-the-failure), not an empty state.

### Color, width & platform

Wording and order carry the meaning; spacing, weight, and hue only reinforce it. Output stays clear with all styling removed, so never rely on color, glyphs, or animation alone.

- Styling goes through shared semantic roles, never raw colors or ANSI codes. Each stream resolves its own capabilities, and no command toggles styling for the whole process.
- **Bold** marks the primary outcome, help headings, and command and option names in help. Values, IDs, paths, prices, and descriptions stay normal foreground.
- **Hue:** green, yellow, and red mark the success, warning, and failure glyphs, and magenta marks help headings. Cyan, at normal weight, marks a complete command offered in an example or next step, on its own line. Templates with placeholders stay foreground, and a command mentioned inline stays foreground in literal backticks, never cyan.
- **Links:** underline the primary destination, such as a deployment's Production URL; other URLs are unstyled. Print URLs in full. OSC 8 may wrap them on supporting TTYs only.
- **Faint** is for the help summary and truly optional metadata, never for context, consequences, prices, IDs, choices, or recovery.
- **Palette:** use the terminal's basic ANSI colors and default background, with no fixed grays, fills, or decoration, and never recolor logs or remote text. The user owns the palette, so no hue is guaranteed enough contrast in every theme.
- **Glyphs** come from one shared set, with ASCII fallbacks for plain presentation and accessible mode, and `...` for `…`. Rich success is `✓`. Plain status rows start at column 0 and state the outcome as a sentence (`Added API_TOKEN`); warnings and errors keep their `Warning:` and `Error:` prefixes.

Color preference, highest first: `--no-color`, a nonempty `NO_COLOR`, `FORCE_COLOR` (`0` means off), then detection per stream, which is off for pipes and `TERM=dumb`.

- Turning color off removes only color; text, glyphs, layout, and streams stay the same.
- Plain presentation, for a pipe or `TERM=dumb`, is unstyled. Forced color may tint only its `Warning:` and `Error:` prefixes, and help stays fully unstyled even when forced.
- JSON, stdout identifiers, secrets, plain values, and tab-separated tables are never styled.

Measure width on the destination stream by display width, with 80 columns when it's unknown.

- Wrap prose to fit, and stay usable at 40 columns, never overlapping or clipping. A value that doesn't fit beside its label moves to the next line.
- The CLI never breaks, shortens, or alters a command, URL, ID, or path; a value longer than the line may soft-wrap in the terminal. Paths display in the platform's native form.

### Time, numbers & units

| Kind | Rule |
| --- | --- |
| Ages | Tables: relative under 7 days (`2m`, `5h`, `3d`), then absolute (`Mar 5, 2026`). Detail views: local time with timezone, plus age where useful (`2026-09-28 14:03 PDT (2h ago)`). Billing, activity, and non-TTY output: ISO 8601 UTC |
| Durations | Compact in tables and status lines (`42ms`, `3.4s`, `2m 10s`, `Ready in 41s`); spaced in sentences (`42 ms`) |
| Cron | The expression and a library-generated reading: `*/5 * * * *  (every 5 minutes)` |
| Money | `$` with no space. Prices, charges, and receipts show cents (`$20.00`, `$0.40`); summaries may round (`$20`) |
| Counts | Numerals with correct plurals (`1 project`, `3 projects`), never `project(s)`. Commas from 1,000; `12.5K` only in summaries, from 10,000; exact numbers in billing |
| Sizes | Decimal units (`MB`); binary units (`MiB`) only for literal memory |

### Writing for the terminal

Use a calm, direct voice. Settle the behavior and its consequence first, then polish the words. Where this manual gives an exact phrase, use it verbatim; it matches the dashboard.

| Surface | Pattern | Example |
| --- | --- | --- |
| Help summary | Imperative verb and object, no period | `List deployments for a project` |
| Question | Short and concrete, ending in `?` | `Which team?` |
| Confirmation | Action, exact object, consequence | `Revoke token deploy-ci? Deployments using it will fail.` |
| Progress | Present participle and `…` | `Building…` |
| Receipt | Past-tense verb and target, in aligned rows | `✓ Deleted         acme-web` |
| Error | Its [class's](https://vercel.com/design/cli#failure-and-recovery-classify-the-failure) formula | `Couldn't find project acme-wbe in acme.` |
| Next step | Benefit and `:`, then the command on its own line | `Stream the build logs:` |

- Name the thing: `Connect your GitHub account`, not `Connect your account`.
- Use active voice, natural contractions, and numerals. Cut filler.
- State the constraint instead of blaming the user: `Names use lowercase letters, numbers, and hyphens.`, not `You entered an invalid name.`
- Name destinations and commands, never `click here` or a bare `Retry`. `Learn more:` appears only as the help section label, followed by its URL.
- Apologize only for a real Vercel fault or disruption.

| Avoid | Words |
| --- | --- |
| Hype | seamlessly, effortlessly, powerful, robust, leverage, blazing, cutting-edge, world-class, utilize, streamline |
| Filler | just, simply, actually, please, in order to, at this time, it's important to note |
| Vague outcomes | successfully, Done!, OK, Unable to, An error occurred, Something went wrong |
| Interjections | Oops, Whoops, Uh-oh, Yay, Heads up |
| Vague prompts | Continue?, Are you sure?, Do you want to… |
| Non-inclusive terms | whitelist and blacklist (use allowlist and blocklist), master and slave (primary and replica), kill (stop, except for literal signals), sanity check, dummy |
| AI cadence | "It's not X, it's Y", "No X. No Y. Just Z.", rhetorical questions, em-dash chains, tricolons for rhythm |

- Use sentence case for prose, questions, help, and statuses, and Title Case for aligned labels, table headers, and help section titles, except `Learn more:`. Nouns keep their [preserved casing](https://vercel.com/design/cli#reference-nouns).
- Periods end sentences, not fragments or labels. Use the Oxford comma, and `…` (one character) for ongoing work. Never use `!` in CLI-written text; it's only the warning glyph. User data and quoted partner messages keep their punctuation.
- Use ASCII quotes and apostrophes, not the dashboard's curly ones: CLI output is pasted into shells and read by consoles that can't render them.
- Write commands, options, paths, and values exactly. Never rewrite JSON keys, telemetry names, third-party literals, or parser values for consistency.

### Next steps & tips

A **next step** follows from current state and appears every time that state applies.

- Offer one, or a single `Next steps:` list of independent actions with required recovery first. Each states its benefit, then the [command](https://vercel.com/design/cli#designing-commands-examples-and-suggestions).
- Check every suggestion against current state. While remote work might still be running, suggest checking status, never retrying.
- Optional steps respect `VERCEL_GUIDANCE_DISABLED` and user settings. Required recovery always shows.

A **tip** (completion setup, a new feature, a better command for what the user just did) is useful once.

- Show a tip at most once per user, recorded in the CLI config directory, and at most one per run.
- Show it only after success, in interactive human mode with no detected agent, after the result, on stderr. Never in CI, non-interactive mode, or `--json`. Tips, update notices, and optional offers share this gate.
- One line starting `Tip:`, which may be faint, then the command on its own line if there is one. A tip never delays, prompts, or changes the exit code, and respects `VERCEL_GUIDANCE_DISABLED`.
- Anything that needs attention every time is a [warning](https://vercel.com/design/cli#failure-and-recovery-warnings), not a tip.

### Machine results

`--json` is the machine interface, and a contract. Every command that reads or changes a resource supports it. stdout then holds exactly one JSON document and a newline. stderr stays empty except for `--debug` output, CI `::add-mask::` lines, and, on failure, the error document. `--follow` switches that invocation to [JSONL](https://vercel.com/design/cli#output-streams-and-other-protocols), and Logs and Traces Search write JSONL rather than one document.

- Mirror the API, with camelCase keys, API enums and timestamps, and stable IDs.
- Guaranteed fields are those in the command's [result schema](https://vercel.com/design/cli#designing-commands-machine-readable-help-and-search). Other API fields pass through unchanged and can change with the API, except secret fields: the shared JSON writer drops any field the schema marks secret, unless the schema marks it as the command's delivered value.
- Key shapes by noun, so top-level fields can be added safely:

| Operation | Shape |
| --- | --- |
| Read one | `{ "project": { … } }` |
| List | `{ "projects": [ … ], "pagination": { "next": "<cursor>" \| null } }` |
| Mutation | `{ "project": { … }, "outcome": "<verb, past tense>" \| "unchanged" \| "accepted" \| "staged" \| "started" \| "planned" }` |
| Bulk | `{ "results": [ { "id": …, "outcome": …, "error"?: { … } } ], "summary": { "succeeded": 2, "failed": 1, "skipped": 0, "unknown": 0 } }` |

```json
{
  "deployment": { "id": "dpl_example", "readyState": "BUILDING" },
  "outcome": "accepted",
  "next": [{
    "command": "vercel deployments inspect dpl_example --wait",
    "argv": ["vercel", "deployments", "inspect", "dpl_example", "--wait"]
  }]
}
```

- Any result may add 2 shared top-level fields: `warnings: [{ "reason", "message" }]` and `next: [{ "command", "argv", "when"? }]`. `when` is the condition for using the command, or a note on filling its placeholders. After a mutation, `next` includes the resource's inspect command, or its list command after `delete` or `revoke`, or when there is no inspect. JSON `next` ignores guidance settings.
- Inside a document, an empty list is `[]` and a known-absent value is `null`, never omitted. Optional top-level fields (`warnings`, `next`, and an error's `userActionRequired`, `retryable`, `requestId`, and `retryAfter`) appear only when they apply. A withheld secret field is omitted, never `null`.
- Pretty-print a stdout document with 2-space indentation when stdout is a TTY, and write compact single-line JSON otherwise. An error document on stderr is always one line. Consumers never depend on whitespace or key order.
- Commands with a result schema accept `--fields <FIELDS>`; an unknown field lists the valid ones.

### Machine errors

With `--json`, a failure writes nothing to stdout and exactly one single-line JSON error document to stderr (pretty-printed here). stderr carries nothing else except `--debug` output and `::add-mask::` lines: no progress, receipts, notices, or hints. Bulk operations and JSONL streams differ: a bulk operation writes its partial result to stdout and no error document, and a JSONL stream keeps the lines it already emitted. In every case, the exit code is the source of truth.

```json
{
  "status": "action_required",
  "reason": "missing_scope",
  "message": "Can't link without a team in non-interactive mode. 3 teams are available.",
  "next": [{
    "command": "vercel link --scope <TEAM>",
    "argv": ["vercel", "link", "--scope", "<TEAM>"],
    "when": "Replace <TEAM> with a team slug from data.teams"
  }],
  "data": {
    "teams": [{ "slug": "acme" }, { "slug": "acme-labs" }, { "slug": "acme-oss" }]
  }
}
```

| Field | Rule |
| --- | --- |
| `status` | `action_required` when the command is blocked before any effect, and input, approval, or a human step would let it run; otherwise `error`, including failures a human must fix after an effect started |
| `reason` | A stable snake_case code from one shared registry of reasons |
| `message` | The human headline without its glyph. Never parse it; its wording changes as copy improves, and `reason` doesn't |
| `next` | Recovery commands, shaped like a result's `next` |
| `userActionRequired` | `true` when a human must approve or perform a step (approval, browser consent, SSO) |
| `data` | Resolved choices and context; `issues` (`[{ "field", "reason", "message" }]`) for validation; the idempotency key or operation ID for `unknown_outcome` when one exists. API and partner messages, and remote values that fail their selector's identifier grammar, go here, never in `message` or `next` |
| `requestId` | When the API provides one |
| `retryable` | `true` only when repeating the command is safe: same target, no duplicated effect. An unknown outcome is `false` unless the rerun reuses a supported idempotency key, and `next` then gives that exact command |
| `retryAfter` | Seconds to wait, when known. A delay, never permission: retrying still needs `retryable: true` |

- The class sets the exit code, never `status`: `missing_scope` exits `2` and `login_required` exits `1`. `canceled` isn't a class; it exits `130` or `128 + n`.
- The error document carries `warnings` too, and is always the last line of stderr.
- With `--json`, a cancel or signal writes one error document with `reason: "canceled"` and the remote state in `data.remote`. Observational follow modes write nothing.
- For destructive and severe operations, `confirmation_required` sets `userActionRequired: true` and puts the preview in `data.effects`. Its `next` entry reads `when: "Only after a human approves: <consequence>"`. An agent must not add `--yes`, or a `--force` that replaces user data, without that approval. For severe work, the continuation command names the resource by its stable ID and the team explicitly. The document may include `data.effectsDigest`; rerunning with `--effects-digest <DIGEST>` fails with `effects_changed` if the target or effects changed since the preview.
- The reason codes in this manual are these JSON values. In human mode, the same conditions print as human errors.

### Streams & other protocols

- JSONL streams write one complete JSON object per line, flushed as each event arrives, with a stable `type` when a stream carries more than one kind of event (`log`, `warning`, `summary`). Never buffer a stream into one JSON array.
- A successful bounded (non-follow) query ends with one summary line: `{ "type": "summary", "recordsReturned", "timeRange", "pagination": { "hasMore", "next" }, "excludedIntervals" }`. `hasMore` says whether more records matched, and `next` is the cursor for `--next`, or `null`. Moving an existing stream to this shape is a [contract change](https://vercel.com/design/cli#compatibility-and-quality-change-contracts-deliberately).
- A stream error ends the stream with one JSON error document on stderr.
- Logs and Traces Search write JSONL with `--json`, and also in any non-interactive run, CI and detected agents included, a long-standing behavior they keep. No other command infers a machine format from the mode.
- Each streaming command documents snapshot versus follow, ordering, time window, reconnect and gap behavior, and when it stops.
- Following a collection (`env list --follow`) prints the current list, then one activity row per change. With `--json`, the first line is `{ "type": "snapshot", "<nouns>": [ … ] }`, and each change is a line with `type` `added`, `updated`, or `removed`. A polling follow mode states its interval in help and honors `Retry-After`.

| Protocol | Rule |
| --- | --- |
| Raw API body | `vercel api` prints the response body exactly; `--include` adds the status line and headers |
| File or bytes | Exact bytes to the `--output` destination or a non-TTY stdout. A TTY gets escaped output unless `--output -`. Status goes to stderr |
| Child process | The child owns stdout, stderr, and the exit code; the CLI never wraps them |
| Plain value | A command that prints one value (a URL, a requested token) prints it alone, with a newline |

## Failure & recovery

### Design errors as recovery

An error tells a person or an agent what failed, why, and exactly what to do next. It never guesses a cause, blames the user, hides partial work, or reports a failed read as empty.

```text
✗ <Headline: what failed, then the known cause.>
  <Detail or issue rows>
  <Context rows>
  <ID rows>

  <Recovery instruction>:
    <command>
```

| Part | Rule |
| --- | --- |
| Headline | The failed action and its exact object, then the known cause, in 100 characters or fewer when possible: `Couldn't find project acme-wbe in acme.` |
| Detail | What the headline can't hold: each validation issue, a code frame, the API's own attributed message |
| Context | Facts needed to act or report: an inferred `Team` or `Project`, `File`, `Limit`, `Docs` |
| IDs | Normal-foreground rows labeled per the [ID labels](https://vercel.com/design/cli#reference-ids), last so they're easy to copy. Resource IDs needed for recovery always show; Request IDs only on Platform, Network, and Unknown outcome errors |
| Recovery | After a blank line, an instruction ending in `:` and the command (or a URL) indented 4 spaces. At most 2 options, likeliest first, or a plain sentence when no command applies. Never "try again" alone: give the command to rerun, or the status command when the outcome is unknown |

- Omit parts with nothing true to say. Outside issue lists and code frames, an error should fit in 8 lines; when trimming, keep required context, consequences, IDs, and recovery.
- A blank line separates the error from earlier output; continuation lines indent 2 spaces.
- Never put a status code in the headline, or show a raw API payload or a stack trace outside `--debug`.

```text
✗ Can't confirm whether the deployment finished. The connection closed while waiting.
  Deployment ID   dpl_9x1Kq2

  Check its status:
    vercel deployments inspect dpl_9x1Kq2
```

### Classify the failure

Every failure has exactly one class. The class fixes the headline formula, the allowed `reason` codes, and the exit code.

| Class | Headline formula | Example reason | Exit |
| --- | --- | --- | --- |
| Usage | `Unknown option --x for vercel <cmd>.` / `Can't use --a with --b.` / `Can't <verb> <object>. <n> <inputs> are invalid.` / `Unknown command <segment> for vercel <path>.` | `invalid_arguments`, `unknown_option`, `unknown_command` | `2` |
| Missing input | `Can't <verb> without <thing> in non-interactive mode.` | `missing_scope`, `confirmation_required` | `2` |
| Authentication | `You're not logged in.` / `Your login expired.` | `login_required`, `token_expired` | `1` |
| Permission | `Only team <role>s can <verb> <object>. Ask a team <role>.` / `As a <role> on <team>, you can't <verb> <object>. A <role> role is required.` / `This token can't <verb> <object>.` | `permission_denied`, `token_scope_required` | `1` |
| Not found | `Couldn't find <noun> <name> in <team>.` | `not_found`, `project_not_found` | `1` |
| Conflict | `Can't <verb> <object>. <Conflicting state>.` | `conflict`, `stale_version` | `1` |
| Limit or plan | `Can't <verb> <object>. <Team> has reached <limit>.` / `Upgrade to Pro to <verb> <object>.` / `Monthly <limit> reached. <What is paused>.` | `plan_limit`, `policy_denied` | `1` |
| Rate limit | `Too many requests to the Vercel API. Try again in <time>.` | `rate_limited` | `1` |
| Network | `Couldn't reach <host>. <Cause>.` | `network_error` | `1` |
| Platform | `Failed to <verb> <object>. <Cause>.` / `Build failed.` | `api_error`, `build_failed` | `1` |
| Unknown outcome | `Can't confirm whether <operation> finished. <Cause>.` | `unknown_outcome` | `1` |
| Partial | `<Verbed> <n> of <total> <nouns>. <m> failed.`, adding `<k> unknown.` when outcomes are unknown; unknown items never count as failed. When every item fails in one class, use that class | `partial_failure` | `1` |
| Verification | `Couldn't verify <URL>. <Known cause>.` The deployment may still be ready; the command reports that state separately | `verification_failed` | `1` |
| Config | `<file>:<line>:<col> <problem>.` / `<file>: <key path> <problem>.` | `invalid_config` | `1` |
| CLI bug | `Unexpected error in vercel <cmd>. This is a bug in Vercel CLI.` | `internal_error` | `1` |

The other reason codes in this manual:

| Class | Codes |
| --- | --- |
| Missing input | `ambiguous_project`, `missing_project`, `env_key_sensitive`, `not_linked`, `missing_name`, `missing_value`, `missing_environment`, `missing_timeout`, `terms_required` (with `userActionRequired: true` and `data.url`) |
| Usage | `invalid_environment`, `branch_requires_preview_only`, `missing_command`, `agent_cannot_switch` |
| Authentication | `sso_required` |
| Permission | `scope_not_accessible` |
| Conflict | `owned_by_another_team`, `env_file_exists`, `price_changed`, `port_in_use`, `stale_link`, `effects_changed` |
| Platform | `cli_outdated`, `spawn_failed` |
| Unknown outcome | `timeout`, for any wait that reached `--timeout` |
| Canceled (exit `130`, or `128 + n` for a signal; not a class) | `canceled`, in JSON only |

- **Verbs:** `Can't` for a present blocker, `Couldn't` for an attempt that failed, `Failed to` for a system or build failure.
- **Permission:** name the blocked action and object, and route to whoever can resolve it. When the API returns the user's role and a recommended role, say both: `As a viewer on acme, you can't update example.com. A developer role is required.` Never infer a role from a list of roles.
- **Plan:** route to the upgrade. Enterprise-only features route to sales.
- **Owned by another team:** a publicly identifiable resource (a domain) owned elsewhere is a Conflict (`owned_by_another_team`), not a permission error, because the user's own team owners can't resolve it. The error says so, then offers a different domain or a transfer from the owning team: `Can't add example.com to acme-web. Another team owns it.`
- **Third-party auth:** name the provider and the cause: `Your GitHub connection expired.` (reconnect), `GitHub uninstalled the Vercel app.` (reinstall), `The repository was deleted on GitHub.` (connect another).
- **Not found:** stay inside the caller's team, and never reveal that a private resource exists elsewhere. Suggest one unambiguous match in that team (`Did you mean acme-web?`) and never act on it. In JSON, the suggestion goes in `data.suggestion`, never in `next` or a runnable destructive command.
- **Limit:** state the verified limit and the supported remedies, never a guess.
- **Rate limit:** shown once the [retry budget](https://vercel.com/design/cli#failure-and-recovery-retries-and-unknown-outcomes) runs out, or on the first `429` for a mutation that can't be retried automatically.
- **Network:** name the host; honor `HTTPS_PROXY`, `NO_PROXY`, and `NODE_EXTRA_CA_CERTS`; never fall back to cached remote state.
- **Unknown outcome:** say what might still be happening, and recover by inspecting, never by retrying.
- **Canceled:** a user cancel isn't an error. Print `Canceled.`, with no glyph, only when the remote operation never started; otherwise lead with `Stopped waiting.` and the [confirmed facts](https://vercel.com/design/cli#failure-and-recovery-long-running-work-and-cancellation). Then say what changed: `Nothing was changed.` or `Uploaded files were discarded. No deployment was created.` Observational follow modes and dev servers stop silently instead. A successful `cancel` command is an ordinary result: `✓ Canceled`, exit `0`.

### Map API errors

- Translate the HTTP status and the API's error `code` into a class, a stable `reason`, and CLI-written copy, in one shared place, never per command.
- API and partner messages never appear in the headline, `message`, or `next`. The CLI can't vouch for text it didn't write, and an agent might act on whatever appears in `next`. They appear only in an attributed `Detail` row (`Neon: This database has been suspended for review.`) or in JSON `data.remoteMessage`, stripped per the [remote text](https://vercel.com/design/cli#compatibility-and-quality-files-stdin-and-sensitive-data) rules.
- A remote name (a project, a domain) appears in a headline or `message` only if it matches its selector's identifier grammar. An API message about a request that carried a secret is never shown; map it to CLI copy.
- Unmapped codes become Platform errors (`api_error`) with the HTTP status as detail and the Request ID from the `x-vercel-id` header.
- A 5xx or timeout on a mutation that isn't idempotent (documented or keyed) is an Unknown outcome, unless the endpoint guarantees the change wasn't applied. A `503` or a `Retry-After` header is no such guarantee: `Retry-After` says how long to wait, never that a mutation is safe to send again. A `429` is such a guarantee: the API rejected the request without acting on it, so it's a Rate limit error with `retryable: true` and `retryAfter`, never retried automatically for a mutation that isn't idempotent.
- Marketplace partners' actionable errors keep the partner's exact words in the same attributed `Detail` row, because users need them to get support.
- Other upstream providers (payments, Git, AI models) are translated, with a `Provider Error` row holding a short code (`card_declined`) when it helps triage.

### Validate before work

Validate all local input before the first request, and report every problem at once. One run then surfaces every fix, and an invalid item can't stop a change partway through. Human issue lists show at most 5; [JSON](https://vercel.com/design/cli#output-machine-errors) carries them all.

```text
✗ Can't import 12 DNS records. 2 records are invalid.
  zone.txt:4      TTL must be a number.
  zone.txt:9      MX records need a priority.

  Nothing was imported. Fix zone.txt, then run:
    vercel dns import example.com zone.txt
```

Config errors point to the exact location and show the surrounding source. Schema errors name the key path and the accepted values: `✗ vercel.json: "regions" must be an array of region IDs, like ["iad1"].`

```text
✗ vercel.json:14:5 isn't valid JSON. Expected "," or "}".
  12 │   "rewrites": [
  13 │     { "source": "/a", "destination": "/b" }
  14 │     { "source": "/c" }
     │     ^
```

Usage errors name the unknown or conflicting token, suggest at most one close alternative without running it, and point to help.

```text
✗ Unknown option --prdo for vercel deploy. Did you mean --prod?

  See all options:
    vercel deploy --help
```

### Error exemplars

```text
✗ Your login expired.
  Team            acme

  Log in again:
    vercel login
```

```text
✗ Can't create project acme-api. acme has reached the Hobby limit of 200 projects.

  Find projects to delete:
    vercel projects list
  Or upgrade to Pro for more projects:
    vercel billing buy pro
```

```text
✗ Unexpected error in vercel env add. This is a bug in Vercel CLI.
  Error           TypeError in src/commands/env/add.ts
  Version         Vercel CLI 60.1.3, Node.js 22.9.0, darwin-arm64

  Review the `--debug` output, which includes team and project names, then report it:
    https://github.com/vercel/vercel/issues/new?template=bug_report.yml
```

A bug report link never includes argv values, paths, or tokens, and the `Error` row never includes text derived from input.

### Warnings

A warning is a nonfatal condition that needs attention; routine information isn't one.

- A warning uses the `!` gutter, states the consequence, and gives the fix: inline when it's a short sentence (`Use --json.`), otherwise in the recovery layout.
- Deprecations name the replacement and print at most once per run: `! --format json is deprecated. Use --json.`
- Human warnings go to stderr. With `--json`, they go in the document's `warnings` array, or as `type: "warning"` lines in JSONL.
- Never print both a warning and an error for the same condition.
- Usage thresholds each include the command to raise the limit or upgrade:
  - At 80%: `! You've used 80% of your monthly <limit> (<current> of <max>).` (`usage_threshold`)
  - At 100%, a blocked command fails as Limit or plan (`plan_limit`): `Monthly <limit> reached. <What is paused>.`
  - In billed overage: `! You've exceeded your monthly <limit>. Additional usage is billed at <rate>.` (`usage_overage`)

| Reason | Condition |
| --- | --- |
| `deprecated` | A deprecated command, option, or alias was used |
| `usage_threshold` | 80% of a monthly limit is used |
| `usage_overage` | Usage is past the limit and being billed |
| `stream_gap` | A follow mode reconnected and might have missed events |
| `link_team_mismatch` | The directory's link uses a different team than the current team |
| `link_tracked` | Git tracks the `.vercel/` directory |
| `link_pin_mismatch` | An environment pin disagrees with the directory's link |
| `insecure_version` | The activated CLI version is below the minimum secure version |
| `limit_clamped` | `--limit` exceeded the command's cap and was clamped |
| `env_file_tracked` | Git tracks, or doesn't ignore, an env file the CLI wrote |
| `env_var_skipped` | A loader or CLI-control variable was withheld from a child process |
| `port_fallback` | The default port was taken, so the next free one was used |
| `secret_not_returned` | A Secret's value wasn't returned and was left unfilled |

### Retries & unknown outcomes

- Every finite operation has request timeouts, a retry budget, and a limit on total waiting, including polls that keep returning a non-terminal state.
- Retry only transient failures, with backoff, honoring `Retry-After`. Reads (`GET`, `HEAD`) retry automatically.
- Retry a mutation only when replaying it is safe: the endpoint documents idempotency, or the API supports an idempotency key and the request reuses it, with the same payload, on every attempt. Otherwise a 5xx or a network failure after sending is an unknown outcome, because the server might already have applied the change; inspect before repeating.
- In human mode, print `Retrying automatically…` while retrying. When the budget runs out, the error keeps its class's headline and adds a `Retries` detail row with the count.
- Never answer an unknown outcome with a blind retry. Keep the operation's identity and a status command.

### Partial & staged work

- Bulk operations report what succeeded, failed, was skipped, or is unknown; whether the operation is atomic; and which subset is safe to retry. A partial result exits `1`.

```text
✗ Removed 3 of 5 DNS records. 1 failed. 1 unknown.
  rec_8aj2        Record not found.
  rec_91kd        The request timed out.

  Check whether the timed-out record was removed:
    vercel dns inspect rec_91kd
```

- `update` distinguishes omitted (unchanged), explicitly empty (cleared), and new values. Read-modify-write uses version or ETag conditions; a conflict means re-read, never a blind retry.
- Staged changes report `✓ Staged <noun> <name>` with a `Version` row, then the diff, publish, and discard commands (`outcome: "staged"`, plus `version`).
- `diff` prints one change per row, prefixed `+`, `-`, or `~`, and exits `0` whether or not anything differs. `--exit-code` makes it exit `1` when something does, for CI drift checks; that's a result, not an error, so the diff stays on stdout and no error document is written. `--json` returns `{ "changes": [{ "id", "change": "added" | "removed" | "updated", "before", "after" }], "version" }`.
- `publish` previews every staged change with who made it, and publishes only the version it previewed; a newer one is a `stale_version` conflict. It's destructive when it blocks or challenges traffic. Never publish unrelated staged work as a side effect.
- Server-side schedules (a progressive rollout) report the start, `✓ Started` and `outcome: "started"`, never their final state.
- Moves that take something from another resource (an alias or domain from another project) show a `Previous` row and return `previous<Noun>` in JSON.
- `--dry-run` does the reads and validation, performs no mutation, and says what would change. It never prompts and needs no `--yes`, still validates a given `--amount` (`price_changed`), and on a staging command previews the stage without staging. In JSON: `outcome: "planned"`, with the `changes` array from `diff`.
- Make no-ops explicit: `example.com is already assigned to acme-web.` (exit `0`, `outcome: "unchanged"`).

### Long-running work & cancellation

- Respond within 100 ms of starting slow work. A spinner names its phase (`Uploading files…`); a progress bar needs a real total; percentages are never invented.
- Animate only with rich presentation in interactive mode. Otherwise, print each phase change once as a plain line, and for byte transfers a line at each further 10%, or after 30 seconds without one, whichever comes first, at most once every 10 seconds: `Uploading big.mp4… 800 MB of 2.0 GB (40%)`. Progress clears before results, prompts, and errors, never spins over logs or child output, and parallel work shares one indicator.
- Past its usual duration, work says so once: `Still <verb>ing. Most <operations> finish within <duration>.` When it's waiting on something external, name it: `Waiting on DNS propagation. This can take up to 48 hours.` Never `This might take a while`. A resource ID or status link may appear as soon as it exists, so people can follow the work; a Request ID appears only on failure.
- Waiting commands support `--timeout` and state its default.
- `--no-wait` returns once work is accepted, reports it as accepted, and gives the ID and status command; it never claims completion. Inspect commands for async resources support `--wait` (with `--timeout`) to block until a terminal state. A compound operation reports only the phases that completed.
- `--wait` exits as the originating command would: a failed build is `build_failed`, exit `1`. A wait that reaches `--timeout` before a terminal state exits `1` with `reason: "timeout"` and the status command. `--timeout` with `--no-wait` is a usage error.
- Follow modes (`--follow`, dev servers) run until stopped and never buffer an unbounded stream. After a disconnect they print `! Lost connection. Retrying automatically…` once, and reconnect with backoff capped at 30 seconds. After reconnecting, they warn once about the gap (`stream_gap`), with a command for the missing window. After 5 minutes without reconnecting, they end with a Network error. With `--json`, the reconnect notice is omitted and the `stream_gap` warning is still emitted.
- An observational `--follow` (one that only watches, such as a log stream) accepts `--timeout`, and reaching it exits `0`, because observing for that long was the task. A follow that waits on work it started follows the `--wait` timeout rule above. In CI, `--follow` without `--timeout` fails as Missing input (`missing_timeout`).
- The first Ctrl+C stops local work and restores the terminal; a second exits immediately. `SIGTERM` and `SIGHUP` get the same bounded cleanup.
- After a cancel, a timeout, or an unknown outcome, report only what's confirmed about the remote work: it never started (`No deployment was created.`); it exists, with its ID, last confirmed state, and whether it may continue (`Deployment dpl_9x1Kq2 continues building. Check it with:` and the command); or its creation is unknown (`Can't confirm whether a deployment was created.`). JSON errors carry this as `data.remote`: `none`, `finished`, `may_continue`, or `unknown`. Recover by inspecting the known ID, or by listing when there isn't one, never by resubmitting. Never claim remote work was canceled because local waiting stopped.

### Exit codes

| Code | Meaning |
| --- | --- |
| `0` | The promised operation completed: success, an empty result, a no-op, acceptance under `--no-wait`, help, or version |
| `1` | Operational failure, as set by the error class; also `diff --exit-code` finding changes |
| `2` | Usage error or missing input, as set by the error class |
| `130` | Canceled by the user: Ctrl+C (`SIGINT`), Escape, or declining a confirmation the operation needs |
| `128 + n` | Terminated by signal `n` (`SIGTERM` exits `143`, `SIGHUP` exits `129`), whether the CLI or its child received it |
| Child code | A command that runs a child (`env run`, extensions) exits with the child's code when the child exits on its own |

- Never exit `0` after a failure because output was printed.
- Declining an offer after the result keeps the result's exit code and prints no `Canceled.`
- A closed stdout (`| head`) ends a read or stream quietly: exit `0`, no error output. A mutation already sent finishes and exits with its real outcome.
- Post-command work (update checks, telemetry) never changes the exit code.

### Pagination & coverage

- `--limit` is the total number of results wanted. The CLI pages internally to reach it and never fetches more than it needs. Every list supports `--limit`, and `--next` when more results can exist.
- A value above the documented cap is clamped, with a `limit_clamped` warning, so a script keeps working when the cap changes. A non-integer or a value below `1` is a usage error. Help documents the default order, default limit, cap, and which filters run on the server.
- When more results exist, return `pagination.next` in JSON and a `Next page:` command in human output. `next` is an opaque, CLI-owned string or `null`, never the API's pagination object.
- Filters, order, team, project, and time window stay fixed across pages. A cursor is bound to them: an omitted filter inherits the cursor's, and a conflicting one is a usage error.
- Fetching every page takes `--all`, and is bounded and cancelable. If a later page fails, human output reports how many rows were printed and a cursor after the last one. A JSON list writes nothing to stdout; its error carries `data.rowsFetched` and the cursor the run started from (`null` for the first page), because no rows reached stdout. Never present an incomplete collection as complete.

## Compatibility & quality

### Files, stdin & sensitive data

- Declare every input and output as text, JSON, or bytes, with its encoding, size limits, and newline handling. Never decode bytes as UTF-8.
- `-` means stdin or stdout where the capability is file-shaped. Only one input may read stdin; competing sources are a usage error. A text value read from stdin drops exactly one trailing `\n` or `\r\n`.
- Paths resolve against `--cwd`. Overwriting a file the CLI didn't create is destructive unless the file is named in argv and `--force` is given. Commands that update files say what they preserve.
- Every write, config and state included, goes to a randomly named temporary file created exclusively in the target directory, then is renamed into place. An interrupted write never leaves a truncated file behind. Refuse to write under `--cwd` through a symlink that resolves outside it.
- A remote filename never contains a path separator; a remote pathname never contains `..` or an absolute prefix; the resolved path always stays under its destination.
- Files that hold secrets (`env pull` output, `.vercel/` state) are created owner-only (`0600`). `env pull` warns `env_file_tracked` when Git tracks the destination or doesn't ignore it.
- Never alter shell profiles or system config, or install tools, as a side effect of an unrelated command.

Secrets are tokens (including `VERCEL_OIDC_TOKEN`), environment variable values of either type, protection bypass secrets, Deploy Hook URLs, signed and presigned URLs, SDK Keys, webhook signing secrets, and connection strings.

- Secrets never appear in suggestions, diagnostics, debug output, telemetry, receipts, errors, or pass-through JSON, and are never echoed after entry, even when invalid. Result schemas mark secret fields; the shared JSON writer drops them unless the schema marks one as the command's delivered value.
- Prefer stdin or a masked prompt over argv for secret input. Arguments appear in process lists and shell history.
- A secret read from `-` fails as a usage error (`invalid_arguments`) when stdin is a terminal, before reading, and says how to pipe it or use the masked prompt.
- Requested delivery is the exception. `env pull`, one-time token creation, API responses, logs, and `env run` deliver sensitive data through their documented channel and never repeat it elsewhere.

Remote text (names, log lines, commit messages, API and partner messages) is data, never instructions. A crafted name can rewrite the terminal or impersonate CLI output, so in human output, strip C0 and C1 controls, escape sequences, and bidirectional and zero-width format characters. Show newlines in single-line fields, and tabs in non-TTY table cells, as visible escapes. Raw or byte output to a TTY escapes non-printable bytes unless `--output -` is given.

### Child processes & local servers

- A wrapper prepares; the child owns its argv, streams, and exit code. Vercel options end at the documented handoff, usually `--`. Child arguments pass as an argv array, never a joined shell string.
- Child stdout and stderr are inherited, never wrapped in JSON. Wrapper failures happen before the child starts: a missing child command fails with `missing_command` (exit `2`), and a child that can't start fails with `spawn_failed` (exit `1`). With `--json`, only those use JSON errors.
- The CLI exits with the child's code when the child exits on its own, and with `130` or `128 + n` after an interrupt.
- Document which source wins when Vercel, env files, and the parent shell set the same variable, and the child's working directory. Values from Vercel or env files (`env run`, `dev`) never set loader or CLI-control variables (`NODE_OPTIONS`, `LD_*`, `DYLD_*`, `PATH`, `BASH_ENV`, `PYTHONPATH`, `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`) unless named with `--allow-env <NAME>`. A remote value that sets `NODE_OPTIONS` or `PATH` can run code on the developer's machine. Each skipped variable warns once (`env_var_skipped`), without its value.
- Forward signals and stop owned processes with bounded waits. Never leave an orphaned server.
- Dev servers print `Ready on <URL>` on stderr once, before child output; a started server is not a finished build. A taken explicit port is a conflict (`port_in_use`) with the recovery `vercel dev --listen <PORT>`. A taken default port moves to the next free one, with one warning: `! Port 3000 is in use. Using 3001.` (`port_fallback`).

### Configuration files

- The CLI edits `vercel.json` only on explicit request, preserving formatting and unrelated keys. It never edits `vercel.ts`; it prints the change instead.
- `.vercel/` holds CLI state: the link, env pulls, build output. Receipts mention it only when the user must act on it.
- The CLI config directory holds auth and preferences, with owner-only credential files. Concurrent CLI processes never lose each other's unrelated keys.

### Updates & version skew

- Update notices pass the [tip gate](https://vercel.com/design/cli#output-next-steps-and-tips), print at most once a day, never appear with `NO_UPDATE_NOTIFIER`, and take no glyph: `Vercel CLI 61.0.0 is available. You have 60.1.3.`, then `Upgrade:` and `vercel cli upgrade`.
- An upgrade offer is one question after the command finishes. Automatic updates are opt-in, and each is reported once.
- `vercel cli upgrade` detects the install method (npm, pnpm, Yarn, Bun, Homebrew, standalone binary) and uses or prints the matching command, never overwriting a package-manager install another way. It runs package managers from the home directory against the official registry, and verifies standalone binaries against a signed checksum before replacing them.
- `cli versions activate` warns `insecure_version` below the minimum secure version.
- When the API rejects the CLI's version, fail with `cli_outdated`, the minimum version, and the upgrade command.
- Unknown API enum values render raw and never crash.
- Auth, config, and caches stay readable by older and newer versions; a format change ships with a migration.

### CI, telemetry & performance

- CI output stays human unless `--json` is selected, because people read CI logs. Logs and Traces Search keep their documented [JSONL](https://vercel.com/design/cli#output-streams-and-other-protocols). Human mode may emit GitHub Actions annotations (`::error file=…,line=…::`) on stderr for config errors, but never workflow commands that change CI state (`::add-mask::`, environment files) unless an option asks for them. The exception: in GitHub Actions, a command delivering a requested secret first emits `::add-mask::` for it on stderr, in every format, `--json` included.
- Disclose telemetry on the first run: a short notice interactively, or one stderr line in CI and non-interactive human mode, never a prompt. With `--json`, the disclosure waits for the next run without it. Nothing is sent before the disclosure. Opt out with `vercel cli telemetry disable` or `VERCEL_TELEMETRY_DISABLED=1`.
- Telemetry records command paths, option names, failure `reason` codes, and exit codes, so the team sees which failures people hit. It never records values, secrets, file contents, paths, or remote data. Unknown path segments are recorded as `<path>` or `<extension>`. It never slows a command or changes its output.
- Help, version, and local commands start without network access, loading command modules lazily. Help and version should render within 200 ms on a warm cache, with startup time tracked in CI so regressions show. Independent requests run in parallel. Non-essential work (update checks, telemetry) never blocks the command.

### Change contracts deliberately

The contract is everything a script can depend on: command names, option names and meanings, defaults, prompts, output streams, JSON shapes, exit codes, environment variables, config files, non-TTY table headers and column order, and a mutation's bare stdout identifier. Change it only as this table allows, and never break a script silently.

| Change | Requirement |
| --- | --- |
| Human copy, layout, or help text outside the contract | Updated tests; no migration |
| Add a command, option, JSON field, or alias | Tests for the new behavior and for unchanged old invocations |
| Rename a command or option | The old name stays as a deprecated alias (warning reason `deprecated`) for at least one major version; ships in a minor release |
| Change a meaning, default, type, stream, exit code, or JSON shape; remove anything | A major version, release notes naming the replacement, and a deprecation period first where feasible |

- An alias bridges a rename, never 2 meanings. Changing what a token means requires removing it first, so an old script fails loudly instead of silently doing something new.
- A breaking stream or format change may ship a compatibility switch (`VERCEL_COMPAT_<NAME>=1`) that restores the old behavior for one major version.
- Use telemetry to measure a deprecated input's usage, without collecting values.
- Add exit code, stream, and JSON assertions for the new contract.

### Verify public behavior

A snapshot, a metadata entry, or a polished transcript doesn't prove a command works. Test public behavior at the narrowest boundary that exercises it, with no live mutations.

| Changed | Evidence |
| --- | --- |
| Names, parsing | Long and short forms behave identically; aliases; unknown, repeated, and conflicting options; globals at nested depth; `--` handoff |
| Help | Every entry point reaches the same help; unknown paths fail; help works offline before auth; section order; examples parse without being run |
| Context, input | Exact resolution; the explicit target wins; typos never create or fall back; malformed config fails; one stdin owner |
| Output | JSON parses from stdout, and error JSON from stderr; each stream asserted; empty versus missing versus partial; no ANSI in JSON |
| Collections | `--limit` is a total across pages with no over-fetch, clamped with `limit_clamped` above the cap; ordering; first, next, and last page; filters fixed across pages |
| Mutations | Nothing written before validation and consent; exact target; default No; `--yes` versus `--force`; no duplicate on retry; partial report |
| Long-running | Bounded waits, including endless pending; cancellation exits `130`; claimed phases happened; accepted versus ready |
| Files, secrets | Encoding and bytes; overwrite refusal; atomic writes; no secret in incidental output; requested delivery still works |
| Processes | argv with spaces and metacharacters; a child's exit `42` passes through; spawn failure; signals; no orphans; a quiet closed pipe |
| Presentation | Before-and-after transcripts at 40, 80, and 120 columns; long and Unicode values; rich and plain presentation; each stream on its own; gutter and receipt |
| Contracts | Old inputs, JSON shapes, streams, and exit codes, unless the change is an intentional contract change |

- For changes to prompts, streams, format, consent, or progress, test every mode; copy-only changes are exempt. Test success and failure on an interactive TTY, with stdout redirected, with stdin piped, with `--json` on a TTY, with `--non-interactive` on a TTY, in CI with a TTY, and as a detected agent. Assert whether a prompt appeared, what each stream holds, and the exit code.
- Check rich and plain presentation on purpose. Run the styled check with `NO_COLOR` and `FORCE_COLOR` cleared, because a shell that sets `NO_COLOR` hides every styling failure. Then check `--no-color`, `TERM=dumb`, and redirected output, with stdout and stderr captured separately. Stripping the styling must recover the exact text, commands and quotes included. JSON, identifiers, and secrets stay unstyled even when color is forced.
- For new commands and flows, you should also walk and read the transcript of each journey the change touches: first run with no login; an expired token or SSO mid-command; several teams; a monorepo; a GitHub Actions deploy; an agent's discover, act, verify, recover loop; offline; rate limited; a long deploy interrupted; partial bulk failure; a destructive Production change; a broken config file; a CLI older than the API expects; Windows PowerShell; accessible mode; piping into `jq`, `grep`, and `head`; two CLI processes at once.
- Parser, help, and global option changes affect every command; link and input helpers affect every setup caller; output helpers affect every adopter. Trace call sites, test the helper, and test at least one materially different caller.
- Keep exact column tests at the shared layout helper. For wording, assert the meaningful new outcome, and reject stale strings where regressions are plausible.

### Enforce the rules in shared code

Rules that depend on every call site erode. Enforce the high-risk ones in shared code rather than in review:

| Rule | Mechanism |
| --- | --- |
| Mode detection in one place | One resolved mode on the client; a lint banning `isTTY` and `process.env.CI` outside the shared module |
| No prompts in non-interactive mode | A prompt helper that throws a typed missing-input error, which the top level maps to `status: "action_required"` and its Missing input `reason` |
| Streams and JSON purity | One result writer for compact or pretty JSON, secret fields, and warnings; a JSON mode that turns human lines into no-ops; a test helper asserting one stdout document and a clean stderr |
| Help contract | Central help dispatch right after the first parse, before config, auth, and team resolution |
| Color and glyphs | One theme module, and a lint restricting color-library imports to it |
| Consent tiers | A `consequence` in command metadata, and one confirmation helper that renders the preview and produces `confirmation_required` |
| Error classes and reasons | A typed error carrying class, `reason`, `data`, and `next`; one API-error mapper; reasons typed from one registry |
| Mutation retries | A fetch client that retries only `GET`, `HEAD`, and requests marked idempotent (documented or keyed) |
| Signals | One root abort signal wired to `SIGINT`, `SIGTERM`, and `SIGHUP`, passed to every request and child process |
| Formatting | Shared formatters for ages, durations, counts, sizes, and money, used by every command |
| Names | A lint against the [Reference](https://vercel.com/design/cli#reference) tables, and a committed surface snapshot |

### Review the consequence

- Reject a change that breaks a requirement, citing the concrete user consequence and the fix. Don't reject on taste or demand unrelated refactors. An existing inconsistency never excuses a new one.
- Also reject partial changes: prompt or result copy edited without checking the surrounding flow; one command path updated while parallel paths sharing its helper or contract are missed; behavior changed without tests that lock in the new and reject the old.
- Review the real before-and-after output for order, rhythm, repetition, and the next action.
- Record exceptions with the rule, the reason, the user impact, and the test. A rule that keeps needing exceptions gets revised.

## Reference

### Nouns

Use these terms in help, prompts, output, and errors. They set human meaning; JSON keys and values still mirror the API. "Identity" lists the forms a selector accepts, and a mutation prints the first as its [stdout identifier](https://vercel.com/design/cli#output-give-each-stream-one-job). "Never say" lists synonyms banned from copy, option names, and positional names.

- Spelling and casing match the dashboard's glossary, so a term reads the same in the dashboard, docs, and CLI. When they disagree, align them; never fork.
- Terms are Title Case in aligned labels, table headers, and help section titles (`Environment Variable`), and lowercase in sentences (`environment variable`), except for preserved-case names.
- Preserved case, Title Case even in prose: Vercel, Vercel CLI (then "the CLI"), Production, Preview, Development, API Key, SDK Key, Rolling Releases, Global Config, Vercel Blob (then "Blob"), Vercel Sandbox, Vercel Firewall, Vercel WAF, Attack Challenge Mode, IP Blocking, Deployment Protection, Skew Protection, Vercel Queues, Fluid Compute, Observability, Web Analytics, Speed Insights, Spend Management, Hobby, Pro, Enterprise, AI Gateway, Vercel Agent, Vercel Toolbar, Instant Rollback, Deployment Checks, Deploy Hook, Marketplace, Observability Plus, Speed Insights Plus, Vercel Connect, Vercel Container Registry, Vercel Token Pass, and the environment variable types Config and Secret. Role names are lowercase in prose (`team owners`, `a viewer`).
- Command tokens may shorten a noun (`routes`, `certs`, `env`, `vcr`); prose always uses the full noun ("routing rule", "SSL certificate", "environment variable").

#### Account & context

| Term | Meaning | Identity | Never say |
| --- | --- | --- | --- |
| Team | Ownership and access context for shared resources, selected with `--scope` | slug or ID | organization (for a team), org, workspace, account; "scope" in prose |
| User | A person with a Vercel login | username | account (for a person), member (outside team context) |
| Member | A user's membership in a team, with a role | username or email | user (when the role matters) |
| Role | A member's access level: owner, admin, developer, viewer, member, and other team roles | role name | administrator, manager, read-only, observer |
| Token | A credential that authenticates the CLI and API | ID or name (never the value) | key, secret, password; API Key (for tokens broadly) |
| Context | Resolved user, team, project, and local settings for one invocation | none | scope |
| Organization | A group of teams managed and billed together through its root team (`vercel organizations`) | slug or ID | org, enterprise, workspace, account |
| Child team | A team in an organization, billed through the organization's root team | slug or ID | sub-team, subteam, member team |

#### Projects & deployments

| Term | Meaning | Identity | Never say |
| --- | --- | --- | --- |
| Project | Persistent resource holding settings, domains, and deployments | name or ID | app, site, repo, repository |
| Deployment | One deploy result with its own ID, URL, and lifecycle | URL (stdout) or ID | build, release, dpl; "deploy" as a noun |
| Build | Work that produces deployment artifacts | ID | deployment, compile |
| Environment | Production, Preview, Development, or a custom environment | slug | target, stage; prod, live, dev, local (in prose) |
| Custom environment | A named project environment, managed with `vercel environments` | slug or ID | target (in prose), stage |
| Environment Variable | Named configuration value scoped to environments and optionally a branch | name | env var, EnvVar; "secret" for any variable that isn't the Secret type |
| Config / Secret | The 2 environment variable types | none | sensitive, plain, encrypted |
| Link | Association between a local directory and a project | directory | connection |
| Root Directory | Project setting for where source lives in the repository | path | project path |
| Git repository | The repository a project is connected to | repository URL | repo (in labels), git repo |
| Deploy Hook | A URL that triggers a deployment from a Git branch | ID | deployment hook, webhook |
| Rolling Releases | Staged traffic rollout between deployments | project | canary release, gradual rollout |
| Instant Rollback | Returning Production to a previous deployment. Verb "roll back"; noun "rollback" | project | revert, roll-back |
| Deployment Checks | Checks that must pass before a deployment is promoted | name or ID | test, gate, precheck |
| Service | One part of a multi-service project, configured in `vercel.json#services` | name | app, microservice |
| Daemon | A long-running container in a project, configured in `vercel.json#daemons` | name | worker, background service |
| Daemon instance | One running copy of a daemon in a region | ID (`dmni_…`) | replica, pod |

#### Domains & routing

| Term | Meaning | Identity | Never say |
| --- | --- | --- | --- |
| Domain | A registered or external domain and its ownership and configuration | domain name | site, URL |
| Apex domain | A domain without a subdomain (`example.com`) | domain name | naked domain, root domain, bare domain |
| Registrant | The person or company a domain is registered to, its legal owner | none | owner (for the registration), WHOIS contact |
| Alias | A hostname assigned to a deployment | hostname | domain (for the assignment); say "command alias" for alternate command names |
| DNS record | A typed DNS entry for a domain | ID | entry, alias |
| SSL certificate | A certificate for a domain (`vercel certs`) | ID | TLS certificate, cert (in prose) |
| Bulk Redirects / redirect | The feature / one rule that sends a request to another URL | source path | redirect rule, route |
| Project Routing Rules / routing rule | The feature / one request-routing rule (`vercel routes`) | name or ID | route (in prose), redirect |
| Vercel Firewall / firewall rule | The product / one custom Vercel WAF rule | ID | the firewall (on first reference), filter |
| IP Blocking / IP block | The feature / one blocked IP address or range | IP or ID | ban, blacklist, IP blocklist |
| Attack Challenge Mode | Firewall mode that challenges all visitors (`firewall attack-mode`) | none | attack mode, challenge mode |
| System bypass | A firewall exception that skips system mitigations | ID | allowlist entry |

#### Data, storage & platform products

| Term | Meaning | Identity | Never say |
| --- | --- | --- | --- |
| Vercel Blob / Blob store / blob | The product / the container / one stored object | store ID / pathname | bucket, file (for the blob) |
| Global Config / store / item | The product / a key-value config store / one key | store ID / key | Edge Config (retired name) |
| Storage resource | A store of any kind attached to projects (`vercel storage`) | ID | database (unless it is one) |
| Integration / installation / resource | Offering / its installed connection / a provisioned instance | slug / ID / ID | plugin, app, add-on |
| Connector | A Vercel Connect connection to a third-party service | ID | integration, app |
| Feature flag | The product resource under `vercel flags` | slug | toggle, feature toggle; "flag" for a CLI option |
| Variant | One value a feature flag can serve | name | treatment, version |
| Segment / entity / label / SDK Key | Targeting group / targeted kind / tag / client credential | slug / kind / name / ID | audience, cohort, subject, principal |
| Target | Entity value a feature flag always serves a fixed variant to | entity.attribute=value | override, allowlist, individual target |
| Experiment | A flag-backed experiment with analysis | slug | test, A/B |
| Vercel Sandbox / sandbox | The product / one sandbox instance | ID | session, container, VM, microVM |
| Vercel Queues / queue / message | The product / a named queue / one message in it | name / ID | topic (unless the API calls it one), job |
| Cron job | A scheduled invocation defined in project config | path | scheduled task, job |
| Schedule | A managed schedule resource (`vercel schedules`) | ID | cron (for this resource) |
| Webhook | An endpoint that receives Vercel events | ID | hook, web hook, deploy hook |
| Microfrontends group | Projects composed into one application | name | micro-frontends, zone |
| KMS issuer / key / grant | Signing identity / key version / project permission | ID | certificate, secret |
| Container repository / image / tag | Vercel Container Registry objects | name / digest / tag | repo |
| AI Gateway API Key / virtual model | Gateway credential / model alias | ID / slug | token, gateway token |
| Vercel Token Pass | Team credits for Token Pass models; API Keys opt in at creation with `--token-pass` | team | gateway token pass |

#### Observability, billing & collaboration

| Term | Meaning | Identity | Never say |
| --- | --- | --- | --- |
| Runtime Logs / Build Logs | Function and request logs / a build's output log | request ID | function logs, build output |
| Drain | A destination that receives forwarded logs, traces, Web Analytics, or Speed Insights data (`vercel drains`) | ID | log drain (for drains broadly), forwarder, sink |
| Trace | A distributed trace for one request | trace ID | span (for the whole trace) |
| Metric | A queryable observability measurement | metric ID | stat |
| Activity event | An audit event for a user or team, in the activity log | ID | log, event log |
| Alert | A triggered alert (`vercel alerts`) | ID | incident |
| Monitor | The rule that defines when an alert triggers and where its notifications go (`vercel monitors`; the API calls it an alert rule) | ID | alert rule, rule |
| Agent Run | One execution in Vercel Agent, with its own ID and state | ID | session, job |
| Agent skill | A directory of instructions and references (`SKILL.md`) that a coding agent loads for a task | name | plugin, prompt, rule |
| Comment / thread | One Toolbar comment / a thread of comments on a page | ID | note, message |
| Spend Management / spend amount | The team feature that limits on-demand spend / the configured limit (a budget). AI Gateway calls its limit a "monthly spend limit" | ID | spend cap, quota |
| Budget alert | A Spend Management threshold reached in a billing cycle (`vercel billing budgets alerts`); not an Alert | ID | alert (alone), notification, history |
| Usage | Metered consumption and cost for a billing period | none | bill, stats |
| Plan | Hobby, Pro, or Enterprise | plan name | tier (in prose), subscription (for the plan) |
| Feedback | Bugs, friction, and feature requests sent to the Vercel team | none |  |
| Incident / component | A disruption reported on the Vercel status page / one monitored part of the platform there (`vercel health`) | ID | outage (for an incident); service (for a component) |
| Billing plan | An integration's pricing option, or how a child team is billed in an organization (`platform` or `enterprise`), selected with `--billing-plan` | slug | plan, tier |

To add a noun, confirm it's a distinct concept in both the API model and the dashboard glossary, add a row with its meaning, identity, and banned synonyms, and use it everywhere at once.

### IDs

Label an ID by what it identifies, matched to its prefix, and never label a prefixed ID `Request ID`. An ID without a row here is labeled `<Noun> ID`; add its prefix row in the same change.

| Label | Prefix | Identifies |
| --- | --- | --- |
| `Deployment ID` | `dpl_` | A deployment |
| `Build ID` | `bld_` | A build |
| `Integration ID` | `cfg_` | An integration installation |
| `Organization ID` | `org_` | An organization |
| `Key ID` | `key_` | An API Key; for KMS keys use `KMS Key ID` |
| `Request ID` | none | One API or platform request (`x-vercel-id`) |
| `Run ID` | none | A workflow, cron, or agent run |
| `Monitor ID` | `ar_` | A monitor |
| `Drain ID` | `drn_` | A drain |
| `Record ID` | `rec_` | A DNS record |
| `Trace ID` | none | A trace |
| `Generation ID` | none | One AI Gateway model generation |
| `Daemon Instance ID` | `dmni_` | A daemon instance |
| `Project ID` | `prj_` | A project |
| `Team ID` | `team_` | A team |

### Verbs

In a group, a command's last segment is a verb from these tables or a view leaf; top-level tools and workflows follow the [command grammar](https://vercel.com/design/cli#designing-commands-command-grammar). Each matches the dashboard's verb for the same action. A new verb needs a row in the same change, and a reason no existing verb fits.

#### Reading & changing

| Verb | Meaning | Use instead of |
| --- | --- | --- |
| `list` | Read a collection | `ls` (alias), `show`, `get` (for many) |
| `inspect` | Read one resource in detail | `get`, `show`, `describe`, `info`, `fetch`, `view` |
| `create` | Make a new standalone resource with its own lifecycle (project, team, token, webhook, store, flag) | `add`, `new`, `register` |
| `add` | Put an existing thing into a container (a domain onto a project), or add an entry to a resource's configuration list (environment variable, DNS record, redirect, rule, label) | `create` (for these), `attach`, `install` (except the `install` shortcut) |
| `invite` | Add a person who isn't a team member yet, by email or account | `add` (for people) |
| `update` | Change the specified fields of an existing resource; omitted fields stay unchanged | `edit`, `patch`, `modify`, `change`, `configure` |
| `set` | Assign a value to a named slot, creating or replacing it (`aliases set`, `flags set`) | `update` (when it's an upsert) |
| `rename` | Change a resource's name only | `update --name` (both may exist) |
| `enable` / `disable` | Turn a feature or resource on or off without deleting it | `turn-on`, `on`/`off` |
| `move` | Change a resource's position in an ordered list (`firewall rules move`) | `reorder` |
| `status` | Read the current state of a singleton feature or operation that has no ID | `inspect` (when there's an ID) |
| `check` | Run checks and report each result (`security check`, `cli versions check`) | `verify` (for checks) |
| `search` | Find items by query (`domains search`, `traces search`, `help search`) | `find`, `lookup` |

Verbs that end or remove something are under [Choose verbs by consequence](https://vercel.com/design/cli#designing-commands-choose-verbs-by-consequence), with how to recover from each.

#### Lifecycle verbs

| Verb | Meaning |
| --- | --- |
| `deploy`, `redeploy` | Create a deployment / rebuild a previous one |
| `promote`, `rollback` | Point Production at a deployment / at a previous one (prose: "roll back") |
| `build`, `dev`, `run` | Build locally / serve locally / run a command or job now |
| `link`, `pull` | Associate a directory with a project / fetch project state locally |
| `connect` | Attach an existing resource or external system (`git`, `storage`, integration resources, connectors); undone by `disconnect` |
| `publish`, `discard`, `restore` | Make staged changes live / drop staged changes / return to a previous version |
| `import`, `export` | Bring many records in from a file / write them out |
| `upload`, `download`, `copy`, `push` | Transfer bytes to or from Vercel / duplicate server-side / upload a container image (Docker convention) |
| `archive`, `unarchive` | Retire without deleting / bring back |
| `pause`, `resume` | Suspend and continue serving or processing |
| `scale` | Override a resource's instance counts without changing its configuration |
| `restart` | Stop a running instance and start it again from the same configuration |
| `start`, `approve`, `abort`, `complete` | Start a rolling release / advance its current stage / stop it and return traffic / finish it |
| `block`, `unblock` | Add an IP block / remove one |
| `purge`, `invalidate` | Remove cached content / mark it stale |
| `buy`, `renew`, `verify`, `claim`, `transfer-in` | Domain and marketplace lifecycle; `transfer-in` starts moving a domain's registration to Vercel |
| `reply`, `resolve`, `reopen` | Reply to a comment thread / mark it resolved / reopen it |
| `open` | Open a resource in the browser (dashboard, Toolbar) |
| `capture` | Record a new sample of live behavior (a trace) |
| `activate` | Make one version the active one among several (KMS keys, CLI versions) |
| `accept` | Accept legal terms |
| `issue` | Have Vercel issue a certificate or credential |
| `diff` | Compare 2 versions, or staged changes with live |
| `apply` | Apply a batch of changes (from input or several options) in one operation |
| `split`, `rollout` | Set a flag's weighted variant split / progressive rollout |
| `prepare` | Generate build-time artifacts (flag definition fallbacks) |
| `switch` | Change the current team |
| `request` | Ask for access to a team (`teams request`) |
| `login`, `logout`, `signup`, `whoami`, `setup`, `upgrade`, `init` | CLI session and setup (prose: "log in", "log out", "sign up") |

- A read-only leaf that prints one derived view or value may be named for it rather than a verb: `firewall overview`, `firewall bot-management`, `teams sso`, `metrics query`, `metrics schema`, `projects access-summary`, `integrations balance`, `integrations discover`, `integrations categories`, `integrations guide`, `activity types`, `flags override`, `flags evaluations`, `flags datafile`, `connectors discover`, `connectors categories`, `agent-runs trace`, `agent-runs projects`, `ai-gateway models endpoints`, `ai-gateway leaderboard *`, `monitors schema`, `domains price`, `billing usage`, `billing contract`, `cli versions installed`, `cli completion`.
- A leaf that issues a credential may also be named for what it returns (`projects token`, `connectors token`, `blob presign`, `blob signed-token`), but issuing is never a read: it follows the [credential rules](https://vercel.com/design/cli#interaction-and-consent-authentication-and-browser-handoffs), and its value is a secret.
- `status` may follow a lifecycle verb to read that operation's state (`deployments promote status`, `deployments rollback status`). It behaves like `inspect`: no remote side effects, and `--json` support.
- Workflow leaves name what the workflow applies to: `setup mcp`, `setup gateway`.

### Selectors & value hints

Name an identity the same way everywhere. Positionals and value hints use the noun, never its type or API field name. The rules for selectors are in [Resource identity](https://vercel.com/design/cli#designing-commands-resource-identity).

| Identity | Option | Positional / value hint | Never |
| --- | --- | --- | --- |
| Team | `-S, --scope` (global) | `<team>` / `<TEAM>` | `--team`, `--org`, `--slug` for a team |
| Project | `-p, --project` | `<project>` / `<PROJECT>` | `<app>`, `<name>`, `projectId`, `NAME_OR_ID`, `PROJECT_ID` |
| Deployment | `--deployment` | `<deployment>` / `<DEPLOYMENT>` | `--dpl`, `--deployment-id`, `<url\|deploymentId>`, `-d` |
| Environment | `-e, --environment` (prefer a comma-separated list; repetition remains supported for compatibility) | `<environment>` / `<ENVIRONMENT>` | `--target`, `--stage`, `TARGET`, `production\|preview` |
| Git remote | `--remote` | none / `<NAME>` | A URL instead of an existing, case-sensitive local remote name |
| Git branch | `--git-branch` | `<branch>` / `<BRANCH>` | `--branch`, `--trigger-branch`, `-b` |
| User | `--user` | `<user>` / `<USER>` | `--user-id`, `<USER_ID>` |
| Domain | `--domain` | `<domain>` / `<DOMAIN>` | `<hostname>` for a domain |
| Store | `--store` | `<store>` / `<STORE>` | `--store-id`, `storeId` |
| Installation | `--installation` | `<installation>` / `<INSTALLATION>` | `--installation-id` |
| Version | `--<noun>-version` | `<version>` / `<VERSION>` | bare `--version` (the CLI's), `--version-id` |
| Request | `--request-id` | `<request-id>` / `<REQUEST_ID>` | `requestId` |
| Any other resource | `--<noun>` | `<noun>` / `<NOUN>` | camelCase (`ruleId`, `issuerId`), `-id` suffixes, `id` alone when the noun is known, `<name-or-id>` |

Allowed value hints: `<NUMBER>`, `<TIME>` (ISO 8601 or relative), `<DURATION>` (`30s`, `5m`), `<PATH>`, `<FILE>`, `<DIR>`, `<URL>`, `<KEY=VALUE>`, `<CURSOR>`, `<TEXT>`, `<USD>`, `<PERCENT>`, `<JSON>`, `<NAME>`, `<VALUE>`, `<TYPE>`, `<FIELDS>`, `<PORT>`, `<LABEL>`, `<DIM>`, `<DIGEST>`. Never `<String>`, `<N>`, `<COUNT>`, `<MS>`, `<Boolean>`, or any hint not listed here or named for a noun.

### Standard options

When a command supports a concept in this table, it uses that row's option. An option for any other concept (`--level`, `--expand`) is named for its field or noun in kebab-case and needs no row. Never advertise an option before its behavior exists.

| Concept | Option | Rules | Never |
| --- | --- | --- | --- |
| Machine output | `--json` | The only format selector. See [Machine results](https://vercel.com/design/cli#output-machine-results). | `--format json`, `-F`, `-j`, `--output json` |
| Consent | `-y, --yes` | See [Bind consent to consequences](https://vercel.com/design/cli#interaction-and-consent-bind-consent-to-consequences) | `--confirm`, `-c` |
| Replace or rerun | `--force` | See [Bind consent to consequences](https://vercel.com/design/cli#interaction-and-consent-bind-consent-to-consequences) | `--overwrite`, `--allow-overwrite`, `-f` |
| Preview | `--dry-run` | Only with a real preview that performs no mutation | `--plan`, `--preview` |
| Variable injection | `--env <KEY=VALUE>`, `--build-env <KEY=VALUE>` | Repeatable runtime / build values. Never an environment selector. | `-e`, `-b` |
| Time range | `-s, --since <TIME>`, `-u, --until <TIME>` | ISO 8601 or relative (`1h`, `7d`); state the default window and timezone | `--from`, `--to`, `--start`, `--end`, `--date` |
| Result size | `-n, --limit <NUMBER>` | Total results wanted; the CLI pages internally ([Pagination & coverage](https://vercel.com/design/cli#failure-and-recovery-pagination-and-coverage)) | `-l`, `--count`, `--top`, `--per-page`; `--limit` for money |
| Continuation | `--next <CURSOR>` | Opaque cursor from the previous page | `--page`, `--cursor`, `-N`, numeric timestamps |
| All items or pages | `-a, --all` | Selects every applicable item (all projects; every entry for `remove`) or fetches every page (bounded, cancelable); one meaning per command, stated in help | `--team-level`, `--everything` |
| Search | `-q, --query <TEXT>` | Free-text search; define matching | `--search`, `-s` |
| Filters | `--<field> <VALUE>` | One option per filterable field. Prefer comma-separated values for OR within a field. Preserve existing repeated forms for compatibility. `--query` may accept a documented search syntax; filters never do. | `--filter <EXPR>` mini-languages |
| Grouping | `--group-by <DIM>` | Prefer comma-separated dimensions for aggregated results. Preserve existing repeated forms for compatibility. Help lists the valid dimensions and the default | `--by`, `--dimension`, `--split` |
| Follow | `-f, --follow` | Stream until stopped; `--no-follow` where following is the default | `--watch`, `-w`, `--tail`, `--stream` |
| Waiting | `--no-wait`, `--wait`, `--timeout <DURATION>` | See [Long-running work & cancellation](https://vercel.com/design/cli#failure-and-recovery-long-running-work-and-cancellation) | `--wait-timeout` |
| File output | `-o, --output <PATH>` | File or directory destination; `-` means stdout | `--out`. `-o json` or `--output json\|yaml\|table\|text` is a usage error suggesting `--json` |
| Payload input | `--file <FILE>`, `--value <VALUE>`, `--data <JSON>` | `-` means stdin | `--json` as input, `--body`, `--items` for the whole payload |
| Field selection | `--fields <FIELDS>` | Standard on JSON commands with a result schema; an unknown field lists the valid ones |  |
| On/off setting | `--<setting>` / `--no-<setting>` | Boolean presence options | `--<setting> on\|off`, `--enabled`, `--disabled`, `<Boolean>` |
| Name | `--name <NAME>` | Display or resource name when it isn't the positional | `-n` |
| Description | `--description <TEXT>` |  | `-d` |
| Message | `--message <TEXT>` | Free text attached to an action |  |
| Metadata | `--meta <KEY=VALUE>` | Repeatable | `--metadata`, `-m` |
| Label | `--label <LABEL>` | Repeatable | `--tag` for the same concept, `-l` |
| Consent digest | `--effects-digest <DIGEST>` | Guard a rerun against changed effects ([Machine errors](https://vercel.com/design/cli#output-machine-errors)) |  |
| Browser | `--no-open` | Suppress opening a browser | `--open` as a string |
| Money | `--amount <USD>` | Currency amounts | `--limit`, `--budget` for money |

The time-range names under "Never" apply to ranges. An option for a single point in time is named for its field: `flags rollout --start`, `ai-gateway leaderboard --date`.

### Short options

A short option (`-` and one case-sensitive letter) has one meaning and one value type across the CLI, so muscle memory from one command is safe in every other. Short forms are for frequent, low-risk inputs. Options that override safety or change what a command does (`--force`, `--dry-run`) never get one, so a typo can't trigger them. `-y` is the one consent shortcut, because it's universal.

| Short | Long | Type |
| --- | --- | --- |
| `-h` | `--help` | boolean |
| `-v` | `--version` | boolean |
| `-d` | `--debug` | boolean |
| `-t` | `--token` | string |
| `-S` | `--scope` | string |
| `-A` | `--local-config` | string |
| `-Q` | `--global-config` | string |
| `-a` | `--all` | boolean |
| `-e` | `--environment` | string |
| `-f` | `--follow` | boolean |
| `-n` | `--limit` | number |
| `-o` | `--output` | string |
| `-p` | `--project` | string |
| `-q` | `--query` | string |
| `-s` | `--since` | string |
| `-u` | `--until` | string |
| `-y` | `--yes` | boolean |

- Use no other letters. To reserve one, add it to this table in the same change, with a reason it's needed CLI-wide.
- A command accepts a short option only if it supports that input. `-p` means project everywhere, or nowhere.
- A destructive or severe command rejects `-a`; `--all` must be written in full.
- The one exception is `vercel api`, which mirrors `gh api` (`-X` method, `-H` header, `-F` typed field, `-f` raw field, `-i` include headers), so muscle memory transfers.

### Command index

Every public command. "Kind" follows the [grammar](https://vercel.com/design/cli#designing-commands-command-grammar) and [naming rules](https://vercel.com/design/cli#designing-commands-names-shortcuts-and-aliases); leaf actions use the [verbs](https://vercel.com/design/cli#reference-verbs), and `*` marks a default action. Parenthesized command tokens are accepted aliases, relative to the row's group unless written as a full path; other parenthesized text is a note.

#### Workflows & tools

| Command | Kind | Notes |
| --- | --- | --- |
| `vercel [path]`, `deploy [path]` | workflow | Bare `vercel` deploys the current directory. |
| `dev [dir]` | workflow | Local development server |
| `build` | workflow | Local build into `.vercel/output` |
| `link` | workflow | Link a directory to a project. `link add` adds a project in a monorepo |
| `pull [path]` | workflow | Pull environment variables and settings into `.vercel/` |
| `init [example] [dir]` | workflow | Create a project from an example |
| `open` | workflow | Open the linked project in the dashboard |
| `login`, `logout`, `signup`, `whoami` | workflow | Session |
| `setup` | workflow | Set up coding agents: `setup plugin`, `setup gateway` (`vercel ai-gateway setup`), `setup skills`, `setup mcp` (`vercel mcp`) |
| `skills [query]` | workflow | Discover agent skills for this project |
| `token-pass [setup]` (`tokenpass`) | shortcut | `ai-gateway coding-agents setup --token-pass`; runs setup when bare and forwards its options |
| `agent init` | workflow | Generate an `AGENTS.md` |
| `upgrade [version]` | shortcut | `cli upgrade` |
| `install <integration>` | shortcut | `integrations add` |
| `switch [team]` | shortcut | `teams switch` |
| `list`, `inspect`, `promote`, `rollback`, `redeploy` | shortcut | `deployments <action>` |
| `remove` (`rm`) | alias | Untaught alias of `deployments delete` |
| `feedback [note]` | workflow | Send feedback with a note or `--field <KEY=VALUE>`. No login or project required |
| `api [endpoint]` | tool | Authenticated API requests; `api list` lists endpoints |
| `curl <path>`, `httpstat <path>` | tool | Requests to deployments with protection bypass |
| `bisect` | tool | Find the deployment that introduced a problem |
| `changelog` | tool | Latest product updates; `changelog search <query>` |
| `health` (`status-page`) | tool | Platform status, active incidents, and incidents resolved in the last 7 days, from the public status page |
| `logs [runtime\|requests]` (`log`) | tool | Runtime logs and CDN requests |
| `help [command]` | tool | Help; `help search <query>` finds commands by task |

#### Groups & products

| Command | Leaf actions |
| --- | --- |
| `deployments` | `list`\*, `inspect` (`--logs` prints build logs; `--wait`), `delete` (`remove`), `promote`, `promote status`, `rollback`, `rollback status`, `redeploy` |
| `projects` (`project`) | `list`\*, `create` (`add`), `inspect`, `update`, `rename`, `delete` (`remove`), `pause`, `resume`, `token`; `members list\|add\|remove`; `checks list\|add\|remove`; `access-groups list`; `access-summary`; `protection enable\|disable\|status`; `web-analytics enable\|disable\|status`; `speed-insights enable\|disable\|status`; `observability enable\|disable\|status`; `static-assets dangerously-delete` (`dangerously-delete-immutable-static-asset`) |
| `domains` (`domain`) | `list`\*, `add`, `inspect`, `remove`, `transfer` (`move`), `verify`, `check`, `price`, `buy`, `renew`, `auto-renew enable\|disable`, `search`, `transfer-in` |
| `dns` | `list`\*, `add`, `inspect`, `update`, `remove`, `import` |
| `aliases` (`alias`, `ln`) | `list`\*, `set`, `remove` |
| `certs` (`cert`) | `list`\*, `add`, `issue`, `delete` (`remove`) |
| `env` | `list`\*, `add`, `update`, `remove`, `pull`, `run` |
| `environments` (`targets`, `target`) | `list`\* |
| `teams` (`team`) | `list`\*, `create` (`add`), `switch`, `request`, `sso`; `members list\|invite\|remove` (`invite`) |
| `organizations` (`organization`) | `list`, `inspect`, `create`, `update`; `teams list\|create\|add` (`team`) |
| `tokens` | `list`\*, `create` (`add`), `revoke` (`remove`) |
| `webhooks` (`webhook`) | `list`\*, `create`, `inspect` (`get`), `delete` (`remove`) |
| `crons` (`cron`) | `list`\*, `add`, `run` |
| `schedules` | `list`\*, `create`, `inspect` (`get`), `update`, `delete`, `enable`, `disable`, `run` (`invoke`); `executions list` |
| `deploy-hooks` (`deploy-hook`) | `list`\*, `create`, `delete` (`remove`) |
| `daemons` (`daemon`) | `list`\*, `inspect`, `scale`; `instances list\|inspect\|restart` (`instance`) |
| `redirects` (`redirect`) | `list`\*, `add`, `remove`, `import` (`upload`), `publish` (`promote`), `restore`; `versions list` (`list-versions`) |
| `routes` | `list`\*, `add`, `inspect`, `update` (`edit`), `remove` (`delete`), `enable`, `disable`, `move` (`reorder`), `publish`, `discard` (`discard-staging`), `restore`, `export`; `versions list` (`list-versions`) |
| `comments` | `list`\*, `inspect`, `reply`, `update` (`edit`), `delete`, `resolve`, `reopen`, `open` |
| `drains` (`drain`) | `list`\*, `inspect`, `create`, `update`, `pause`, `resume`, `delete` |
| `alerts` | `list`\* (`ls`), `inspect` |
| `monitors` | `list`\* (`ls`), `add` (`create`), `inspect`, `update`, `remove`, `schema`; `notifications list\|connect\|disconnect\|update` (`link`, `unlink`) |
| `integrations` (`integration`) | `list`\*, `add` (`install`), `remove`, `update`, `open`, `discover`, `categories`, `guide`, `balance`; `terms accept` (`accept-terms`); `installations list` (`installations`) |
| `integrations resources` (`integration resource`, `integration-resource`, `ir`) | `inspect`, `connect`, `disconnect`, `delete` (`remove`), `claim`; `thresholds add` (`create-threshold`) |
| `storage` | `list`\*, `create`, `inspect` (`get`, `status`), `update`, `delete`, `connect`, `disconnect` |
| `blob` | `list`\*, `inspect`, `upload` (`put`), `download` (`get`), `copy`, `delete` (`del`, `remove`), `presign`, `signed-token`; `images upload` (`put-image`); `stores list\|create\|inspect\|delete\|empty` (`list-stores`, `create-store`, `get-store`, `delete-store`, `empty-store`) |
| `global-config` (`edge-config`) | `list`\*, `create` (`add`), `inspect` (`get`), `update`, `delete` (`remove`); `items list\|inspect` (`items`); `tokens list\|create\|revoke` (`tokens --add`, `--remove`); `backups list\|inspect\|restore` (`backups`) |
| `flags` | `list`\*, `create`, `inspect`, `update`, `delete` (`remove`), `set`, `enable`, `disable`, `archive`, `unarchive`, `split`, `rollout`, `override`, `prepare`, `datafile`, `open`, `evaluations`; `targeting enable` (`use-targeting`); `rules list\|add\|update\|move\|remove`; `targets list\|add\|remove`; `segments list\|create\|inspect\|update\|delete`; `entities list\|create\|update\|delete`; `labels list\|set\|add\|remove` (`apply`, `ls`, `rm`); `sdk-keys list\|create\|revoke` (`add`, `remove`); `versions list\|diff` |
| `firewall` | `overview`\*, `status`, `diff`, `publish`, `discard`; `rules list\|add\|inspect\|update\|remove\|enable\|disable\|move` (`edit`, `reorder`); `ip-blocks list\|block\|unblock`; `system-bypass list\|add\|remove`; `system-mitigations pause\|resume`; `attack-mode enable\|disable`; `bot-management`; `alerts list\|inspect`; `persistent-actions list\|inspect`; `traffic list\|inspect` |
| `git` | `connect`, `disconnect` |
| `kms` | `list`\*, `create` (`add`), `inspect`, `update`, `delete` (`remove`), `import`; `keys create\|import\|activate\|revoke` (`add-key`, `import-key`, `activate-key`, `revoke-key`); `grants add\|update\|remove` (`add-grant`, `update-grant`, `remove-grant`) |
| `ai-gateway` | `coding-agents setup`; `api-keys list\|create\|inspect\|revoke` (`add`, `remove`); `budgets list\|inspect\|set\|remove`, `budgets defaults list\|set\|remove`; `models list\|endpoints`; `rules list\|add\|update\|remove`; `virtual-models list\|create\|inspect\|update\|delete\|restore` (`add`, `remove`); `leaderboard apps\|labs\|models\|providers` ("app" here is a third-party application ranked by usage, never a project) |
| `microfrontends` (`mf`) | `pull`; `groups list\|create\|inspect\|delete` (`create-group`, `inspect-group`, `delete-group`); `groups projects add\|remove` (`add-to-group`, `remove-from-group`) |
| `rolling-releases` (`rolling-release`, `rr`) | `inspect` (`fetch`), `update` (`configure`), `start`, `approve`, `abort`, `complete` |
| `connectors` (`connect`) | `list`\*, `create`, `update`, `delete` (`remove`), `connect`, `disconnect` (`attach`, `detach`), `open`, `token`, `discover`, `categories`; `tokens revoke` (`revoke-tokens`); `contacts list\|add` |
| `sandbox` | Delegated to the Sandbox CLI. It owns its tree but follows this registry and renders help in the [standard layout](https://vercel.com/design/cli#designing-commands-help-and-discovery) |
| `vcr` | `list`\* (`ls`), `create` (`add`), `inspect`, `delete` (`rm`), `build`, `push`, `login`, `update` (`config`); `images list\|inspect\|delete` (`rm`); `tags list\|inspect`; `permissions list\|add\|remove` (`clear` is `remove --all`) |
| `ci` | `enable`, `list` (`ls`), `inspect`, `logs`, `grep`, `rerun` |
| `agent-runs` | `list`\*, `inspect`, `trace`, `projects` |
| `metrics [metric]` | `query`\*, `schema` |
| `traces [request-id]` | `inspect`\* (`get`), `search`, `capture` (`create`; shortcut for `curl --trace`); `config list\|set\|remove` (`ls`, `rm`) |
| `activity` | `list`\* (`ls`), `types` |
| `cache` | `purge`, `invalidate`, `dangerously-delete` |
| `security` | `check`\* |
| `billing` | `usage` (`vercel usage`), `contract` (`vercel contract`), `budgets list\|inspect\|set\|remove`, `budgets alerts list`\* (`vercel budgets`), `buy addon\|credits\|pro` (`vercel buy`) |
| `cli` | `upgrade`, `versions installed`\*`\|list\|check\|activate` (`vercel version`, `vercel version use`), `autoupdate enable\|disable`, `telemetry enable\|disable\|status` (`vercel telemetry`), `guidance enable\|disable\|status` (`vercel guidance`), `completion` |

Where groups overlap:

- **Budgets:** Spend Management budgets live in `billing budgets` (team) and `ai-gateway budgets` (gateway monthly spend limits); each help page names the other.
- **Alerts:** budget alerts (`billing budgets alerts`) are Spend Management thresholds reached; `alerts` are observability alerts; `firewall alerts` are Vercel Firewall alerts. Each help page names the others.
- **Schedules:** `crons` are defined in config; `schedules` are managed through the API.
- **Environments:** `env` manages environment variables; `environments` manages custom environments.
- **Storage:** `storage` is the cross-product view; `blob` and `global-config` manage their own stores.
