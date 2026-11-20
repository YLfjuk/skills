---
name: lead-orchestrator
description: Coordinate implementors and their review teams across repositories, track design decisions and dependencies, and prepare delivered changes for the user's merge decision. Use only when the user explicitly invokes lead-orchestrator or asks to use this skill. Review-only requests belong to review-orchestrator.
---

# Lead orchestrator

- **Explicit activation only:** Run this workflow only when the user explicitly invokes `lead-orchestrator` or asks to use this skill. An ordinary request to lead or orchestrate implementation does not activate it.
- **Select by responsibility, not verb:** Do not select this skill merely because the user says "lead" or "orchestrate". Reviewing an existing PR/MR belongs to `review-orchestrator`. Use this skill when responsible for getting changes implemented and through review, including coordinating fixes and dependencies toward delivery.
- **Ownership and context:** Own the overall outcome across one or more workstreams, features, or fixes while implementors own their changes and review orchestrators own their review teams. Coordinate through the current conversation, and do not assume this conversation's project is the repository being changed.

## Establish the assignment

### Context and plan

- Resolve the objective, acceptance criteria, target repositories and host URLs, workspace or remote environment, base branches, existing work, and requested implementor and review models.
- Infer reliable context before asking for missing information.
- Read applicable instructions in each target repository; instructions and permissions from the coordinator's project do not automatically transfer.
- Record a compact plan covering workstream owners, dependencies, design questions, verification expectations, and permission boundaries.

### Team and execution mode

- Use the smallest useful team, and reuse existing authorized work rather than starting competing implementors or review teams.
- Invoking this skill to orchestrate work authorizes creating its team. Prefer separate user-facing threads when the environment supports and permits them, unless the user requests child subagents.

### Capabilities and model policy

- Discover available providers, models, workspace binding, host access, and execution capabilities before launch.
- Honor explicit settings per implementor, review orchestrator, and reviewer instance, including phase-specific design versus coding choices.
- Model precedence:
  1. When available, use the `review-orchestrator` model precedence and `exact`, `preferred`, and `auto` policies.
  2. Otherwise, default to the current model for omitted settings and treat named models as exact unless the user permits substitution.
- Explain any automatic selection briefly. Never silently change an explicit model or execution mode.
- A missing provider blocks only work that needs it.

### Workspace and base binding

- Use supported workspace selection when launching work elsewhere; a repository path in a prompt does not bind a thread to that workspace.
- Verify the actual binding and base before authorizing edits.
- Independent work normally starts from the target's default branch; dependent work may require an exact unpublished parent commit.
- Keep repository, branch, workspace, host URL, and coordinator project separate in the record.

## Keep coordination private and useful

### Context minimization

- Give each team only its assignment, relevant decisions, dependencies, acceptance criteria, permissions, and the lead's return channel.
- Keep briefs and follow-up updates strictly need-to-know for that worker's own actions: omit review-team composition, downstream workstreams, roadmap sequencing, and other behind-the-scenes orchestration details that do not affect what this worker must do.
- Do not provide sibling thread IDs, links, transcripts, or a team directory unless a concrete task requires that information. Prefer a short factual dependency summary with commit or PR evidence.
- Use fresh context rather than inherited coordinator history where the interface permits it.
- Retain peer identities centrally for recovery and user visibility; withholding them from worker briefs does not mean hiding team activity from the user.

### Worker coordination boundaries

Tell every implementor and review orchestrator:

> - Route coordination affecting broader work through the lead.
> - Do not discover, read, message, or start sibling workstream threads.
> - Make routine decisions within your scope. In separate-thread mode, ask the user directly in your own thread when input is needed (needing user input does not require the lead's intervention); in child-subagent mode with no user-facing worker thread, route user questions through the lead.
> - Notify the lead about decisions affecting broader design, scope, dependencies or other workstreams, team blockers, and meaningful status or readiness changes. Before ending a phase or review round, send a brief final status (such as current head and open blockers) to the lead's return channel.
> - Report relevant outcomes rather than routing each conversation through the lead. Batch routine progress; do not announce every edit or wait.
> - Never merge or enable auto-merge without the lead relaying explicit user authorization for this request.

### Review-team routing and host comments

- A review orchestrator may coordinate its own specialist reviewers under `review-orchestrator`, and must apply the sibling-workstream restriction to those reviewers too.
- Internal validation and deduplication remain within the review team. The review team's designated posting owner must put actionable findings, implementation questions, fix requests, and follow-up in code-review comments.
- **Do not relay review findings:** The lead must not relay code-review findings, questions, fix requests, or follow-up to the implementor. An ordinary defect does not require lead involvement merely because it blocks merge. Status reports may link to findings without requesting a relay.
- If posting fails, report the access blocker and retain findings for posting rather than silently switching channels.
- Honor explicit read-only or alternate-routing instructions from the user.

## Preserve decisions and ownership

### Durable checkpoint

Maintain a durable checkpoint outside installed skills and unrelated target checkouts, using available app storage or a user-local file. Record:

- Workstream IDs, scope, target repository/workspace/base, execution handles, actual models, PR/MR URLs, phase, and last meaningful status.
- Accepted decisions, their user or evidence source, superseded proposals, unresolved questions, and dependent workstreams.
- Current implementation head, reviewed heads and coverage, verification results, blockers, host-comment ownership, and monitor registrations.
- Merge authorization, its scope and conditions, and any separate release or publication authorization.
- User-stated standing review or workflow preferences for the session (such as model constraints, preferred specialist coverage, or roster-approval preferences).

### Decisions and authority

- Store decisions as concise contracts and examples, not transcripts.
- Distinguish user-approved behavior, implementor proposals, prototype limitations, and missing agreed functionality. A prototype omission does not reopen an accepted requirement.
- Share relevant decisions with affected owners and confirm receipt for material handoffs.
- Let teams settle local details and ask the user directly about choices that change agreed behavior, public contracts, scope, dependencies, or a user's stated tradeoff.
- Have teams report resulting decisions to the lead when they affect broader coordination; the lead coordinates consequences without becoming a mandatory approval channel.
- Reconcile conflicting answers or superseded requirements before clearing dependent work. Do not turn an unanswered design question into an assumed acceptance criterion.

### Write ownership and holds

- Assign one writer per branch or overlapping change area, and prefer separate supported workspaces for concurrent edits and verification.
- Dependent design can proceed against a named provisional base while coding remains held.
- Keep each workstream within its agreed PR scope; defer unrelated or follow-up capabilities to separate user-approved workstreams rather than bundling them into an active PR.
- State exactly what releases each hold. Treat these as separate events:
  - Design agreement
  - Implementation clearance
  - Dependency merge
  - Release approval

## Implement and review

### Assignment briefs

- Write each brief for its assignment, including relevant accepted decisions, scope and acceptance criteria, target workspace and exact base or review head, dependencies and holds, model settings, permissions, and the lead's reporting channel.
- Pass accessible contracts and evidence rather than assuming workers can read the original conversation or local configuration paths.
- Include the worker coordination boundaries above without sharing sibling identities.
- Tell each implementor to use the `pr-implementor-babysitter` skill when available for author follow-up, subject to the assignment's permission boundaries.

### Launching reviews

- **Wait for a published PR/MR:** Do not queue or launch a review team until the implementor publishes a reviewable PR/MR and reports its URL and head commit.
- Once published, explicitly invoke the `review-orchestrator` skill in the review team's brief (using the target environment's explicit skill-invocation syntax when applicable). Pass the actual request URL, target workspace, relevant accepted design, exact candidate head, model choices, any user-stated standing review terms, and the lead's reporting channel.
- Unless the user already approved an exact roster for that specific PR/MR (quoted verbatim in the brief), let the review orchestrator handle roster approval under the user's stated review terms (standing review preferences do not count as an approved roster).
- Resolve that skill's installed instructions and configuration rather than copying its roster or scorecard into this skill.
- If the `review-orchestrator` skill is unavailable, report the limitation and arrange a supported review alternative with the user; do not claim that workflow ran.

### Independence and PR/MR ownership

- Keep review assessment independent: provide requirements and source evidence without priming reviewers to agree with an implementor's conclusions.
- Assign PR/MR babysitting in the briefs and leave host comments and PR/MR watches to the assigned teams:
  - **Implementor:** monitors incoming feedback, makes fixes, and replies on the host.
  - **Review orchestrator:** owns specialist selection, deduplication, posting, and review follow-up, including rechecks after pushes.
  - **Lead:** tracks the teams' reports and resolves cross-team conflicts and design implications.

### Commit coverage and staleness

- Reviewers report concerns against exact commits. After pushes or rebases, the review orchestrator routes changes to the affected reviewers and batches them into useful review rounds.
- Mark old assessments stale until the review orchestrator confirms coverage of the new head.
- A final readiness claim requires all required scopes accounted for at the current head, including integration effects between workstreams. Passing CI or agreement among reviewers alone is insufficient.

### Non-convergence and cleanup

- When review and fixes stop converging, summarize the disputed evidence and tradeoff for the user instead of spawning more opinions indefinitely.
- Honor user budgets or round limits and preserve blockers when a limit is reached.
- Cancel superseded work through the supported lifecycle; do not leave competing assignments active.

## Prepare the user's decision

### Merge and release gates

- **Default:** Await the user's own review and explicit merge permission. Neither reviewer approval, a readiness score, green checks, nor "continue" authorizes merging. Under this default, do not merge, approve on the user's behalf, enable auto-merge, or permit another team to do so.
- **Explicit exceptions or standing permission:**
  - Record the exact request, reviewed head or scope, and conditions. Permission for one request does not transfer to another.
  - Any externally observable change to public contracts or types, or any substantive change to behavior or design, invalidates prior human review or conditional merge or release permission unless the authorization explicitly covers it.
  - A later or narrower user hold or restriction overrides conflicting parts of an earlier permission; confirm with the user if scope is ambiguous, and do not assume lifting a hold restores an earlier permission unless the user explicitly said so.
  - Refresh host state and checks immediately before any authorized merge.
- **Releases are separate:** Publishing packages, releases, or deployments requires its own authorization; merge permission does not imply it.

### Readiness report

- Present a compact readiness report containing:
  - PR/MR links and heads
  - Delivered behavior and remaining design decisions
  - Review coverage and unresolved disagreements
  - Verification limits and host blockers
  - A recommendation
- Separate technical readiness from human review and merge permission.
- When ready, ask for the user's review or merge decision on that concrete result, and continue authorized fixes and review follow-up while permission remains pending.

### Status and progress reports

- Post concise status and progress reports in the lead's conversation when overall progress changes substantially or something important arises (such as a blocker, a dependency becoming ready, or work waiting for the user).
- Summarize what changed, current workstream status, blockers or pending input, and what happens next.
- When waiting for the user, identify the needed decision or action and link to the team thread or review discussion where it belongs; do not move the conversation to the lead unnecessarily.
- Batch related events, skip repeated unchanged reports, and continue independent work while a team waits.

### Monitoring and recovery

- Have the implementor and review orchestrator report who owns each PR/MR watch and what it covers. Report monitoring as active only after the owner confirms registration.
- First check team threads and host PR/MR state read-only (without posting any message) to verify liveness or reconcile stale status (for example, confirming an implementor is actively working or already picked up posted review feedback). Only if the team is idle, stalled, or missing an expected terminal status, send one concise nudge asking for status or watch ownership—without restating, ranking, or requesting fixes for review findings. If still unconfirmed, report the verified host or thread facts and mark the team's own status as unconfirmed.
- Follow the environment's wait and handback rules; if monitoring is unavailable, state what event needs a new invocation.
- On recovery, reconcile the checkpoint with actual executions, heads, host discussions, and decisions before launching anything again.
- Have the owners stop obsolete monitoring when the work ends.
