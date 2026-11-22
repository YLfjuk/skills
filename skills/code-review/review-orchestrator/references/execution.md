# Execution and review ownership

## Resolve capabilities

- Inspect the current environment's supported model discovery, child delegation, conversation launch, host access, and monitoring capabilities.
- Resolve every reviewer's model and policy before launch. Instances of the same role may use the same or different providers and models. Use documented interfaces and lifecycle rules rather than assuming particular tool names or a harness.
- Use child delegation for `subagents` and separate top-level conversations for `threads`. Select the configured provider and model through a mechanism that supports them.
- If the requested model or execution mode is unavailable, explain the limitation and apply the model selection policy or ask which execution alternative to use. Do not silently switch modes.
- Use the configured host connection for request state and discussions. Register the review URL with the current workspace or conversation if the environment supports or requires it, and report registration failures.
- Respect any scope, communication, or reporting-channel boundaries given in the invoking request, propagating applicable isolation boundaries into reviewer briefs and sending a brief round-end status (reviewed head, open blockers, watch ownership and coverage, and whether user action is needed) when a return channel is specified.

## Reviewer brief

- Include concrete values, the selected role's full definition and procedures, and instance-specific settings in every brief.
- Read any metadata requirements declared by the role, collect the requested values using its defaults and inference rules, and include them in the brief. Ask for required values that cannot be established reliably; independent reviewers may proceed.
- List discovered skill names (and locations when helpful) so reviewers load them directly rather than pasting full skill contents into the brief.
- Reviewers must not need access to the orchestrator's personal role file.

```text
Review [request identifier]({configured full review URL}) as the {Role} Reviewer.
Reviewer instance ID: {stable instance ID}
Selected provider/model: {resolved provider and model}
Host/API connection: {resolved host connection and endpoint, if needed}
Repository/workspace: {repository and intended workspace}
Exact head commit to review: {SHA}
Role instructions: {full effective role definition and procedures}
Role metadata: {values required by the role, if any}
Instance scope and additional instructions: {effective instance settings}
Coordination boundaries: {scope, communication, or isolation constraints, if any}
Available relevant skills: {resolved skill names and locations to load}
Apply pr-review-babysitter when available (using the target environment's explicit skill-invocation syntax when applicable), subject to the posting mode and ownership below.
Discover and apply other available skills relevant to this scope and changed code.
Posting mode: {live, staged, or read-only}
Ownership: {mode-specific ownership text below}
Score criteria relevant to your assigned scope (plus any user-specified criteria: {criteria, if any}) from 1 to 10:
  1 = serious concerns, 5 = mixed evidence or meaningful gaps, 10 = strong evidence with no material concerns (use `unknown` if evidence is insufficient).
For each score, give a brief evidence-based reason; if below 10, note whether the gap reflects open defects, bounded verification limits, or both.
Return the head actually reviewed, per-criterion scores, findings with evidence and useful locations,
overlapping discussions, unresolved concerns, verification limits,
and whether this assigned scope is complete.
Do not approve, merge, close, or implement changes to this request.
Use an isolated checkout for verification that could disturb another workspace.
```

- **Independent second opinions:** Give each instance the same source evidence and let it reach its initial judgment before sharing sibling findings. Reviewers may inspect existing host discussions. Compare results after their initial assessments; independent review does not authorize duplicate host comments.

## Subagents

Include this ownership text:

> Return findings and assessments to the orchestrator. Do not post host comments, resolve host discussions, or register monitoring. The orchestrator owns validation, deduplication, posting, replies and resolution of its findings, and monitoring. This overrides posting and monitoring defaults in any applied reviewer skill. Finish the review round after returning your result.

- **Launch and checkpointing:**
  - Retain execution IDs, instance ID, selected model, role, and reviewed head in the checkpoint.
  - Use a stable retry identifier per instance and round where supported.
  - After a failed or ambiguous launch, inspect existing work before retrying. Avoid starting a second equivalent task for an instance whose round is active; intentional parallel instances of one role are separate work.
- **Follow-up rounds:**
  - On new commits or author responses, re-engage affected instances using the delegation interface's supported lifecycle. Include the original brief, exact new head, prior findings, author responses, user-agreed exceptions, and unresolved objections.
  - If a new task is required, retain its ID as a new round. Use returned execution handles as documented; backing storage or conversation identifiers are not automatically continuation targets.
  - If a new commit arrives mid-round while reviewers are still running: for a small or incremental push, let in-flight reviewers finish on the prior head and run a delta recheck against the new head; for a substantial rewrite or rebase, cancel and restart on the new head when supported and cheaper. Mark prior coverage stale until the latest head is reviewed.
  - In `staged` posting mode, re-validate held drafts against the latest head before posting, mark obsolete or moved drafts rather than posting stale anchors, and obtain renewed user approval if re-validation materially changes a draft's wording, target or anchor, head, or effect.

## Threads

The user must select `threads` to authorize separate top-level reviewer conversations. Include this ownership text:

> You own posting and follow-up for your assigned findings. Verify public visibility and actual callers before claiming a breaking change, calibrate blocking vs. advisory severity against the repo's context, and lead every host comment with the agent-disclosure callout. Check existing discussions and coordinate overlaps with the orchestrator before posting a concern another reviewer owns. Report findings, host IDs, reviewed head, unresolved concerns, completion, and monitor registration status. Respect staged and read-only posting modes when configured: in staged mode, return all drafted host mutations (findings, replies, agreed-exception notes, and proposed resolutions) to the orchestrator for user approval, and post or execute only the exact approved version once the orchestrator confirms approval.

- **Launch and workspace binding:**
  - Create one conversation per reviewer instance, including its role and instance ID in the title and its full brief as the initial prompt.
  - Configure provider, model, and intended workspace through the supported launch interface. Verify inherited workspace settings when the interface lacks a selector; prompt instructions alone do not establish a conversation's workspace binding.
- **Tracking and follow-up:**
  - Retain conversation IDs keyed by instance. Confirm readiness when launch preparation is asynchronous.
  - Inspect existing conversations after ambiguous launch results before retrying. Another instance of the same role is not a duplicate.
  - Continue follow-up through the interface's supported conversation lifecycle with the updated head and prior review context.
- **Coordination:** Maintain overall status without posting a second copy of reviewers' findings. Assign one owner to overlapping concerns. Register coordination monitoring only when needed and give it a distinct purpose from reviewers' finding follow-up.

## Findings and resolution

- **Validation, calibration, and deduplication:**
  - Validate findings against the cited commit, requirements, and current discussion history.
  - Before classifying a change as breaking, verify its public-contract or export visibility and check actual codebase call sites.
  - Calibrate blocking vs. non-blocking severity by impact and plausibility in the repository's operating context—separating merge-blocking defects from low-probability edge cases or speculative hardening—while preserving helpful non-blocking suggestions as resolvable threads clearly labeled as advisory/non-blocking and explicitly passing the decision to the implementor or maintainer (for example, noting that it is optional and up to them to adopt or resolve) rather than suppressing them.
  - Consolidate duplicates with each other and existing feedback, retaining contributing instance IDs and evidence. Agreement between reviewers does not replace evidence; preserve substantive disagreement and explain the decision.
- **Posting, staging, and agreed exceptions:**
  - In `live` posting mode, post one actionable discussion per concern using a resolvable mechanism when available (or track resolution in the checkpoint if the host lacks one), unless a role's procedure requires separate discussions per related location.
  - In `staged` posting mode, hold all host mutations (findings, replies, agreed-exception notes, and proposed thread resolutions) and present the exact drafted batch—including location, head SHA, calibrated severity, agent-disclosure callout, and full wording—plus dismissed/downgraded items for the user to approve, edit, or drop. Clarify whether approval applies to the current batch or follow-up rounds; if unspecified, treat approval as covering only the presented batch, and require renewed approval if re-validation materially changes a draft's wording, target or anchor, head, or effect.
  - When review discussion with the user establishes a non-obvious decision or accepted exception that affects the change (such as accepting a risk or intentionally deferring a concern), post a brief note on the relevant host discussion or PR/MR in `live` posting mode (or include it in the drafted batch in `staged` mode) so future reviewers and the author have the rationale.
- **Agent disclosure:** Lead every agent-posted host comment or review with a brief note callout or admonition (using the host's callout syntax, or a plain leading blockquote when unsupported):
  - **Agent-authored** *(agent decided on or substantially wrote the point)*: `**Agent-authored** by {agent/model identity} ({role or instance, if applicable}).`
  - **Agent-assisted at the user's direction** *(user directed the substance and asked the agent to phrase or post it)*: `**Agent-assisted** by {agent/model identity} at the user's direction (on behalf of {user handle, if known}).`
  - This distinction depends on who directed the substance, even when posted from a shared account. Never imply a human wrote an agent comment.
- **Verification and resolution:** Verify the substance of a fix or explanation before marking a concern addressed. Resolve only discussions owned by the configured posting owner unless otherwise authorized; allow the implementor to self-resolve a thread without waiting only when they applied a suggested diff in full, line for line, covering the entire concern, or when the thread was explicitly marked as a non-blocking advisory left to their discretion (still checking its status and any resulting change on later passes). Do not treat a host's resolved status as proof of a fix. Whenever a newer review pass supersedes an earlier self-owned hold or summary discussion—and before posting a completion summary—audit self-owned resolvable threads and resolve superseded ones with a brief pointer so no stale self-owned thread remains open. Scope completion summaries to finished instances and named commits.

## Monitoring

- **Registration and wake-up:**
  - Respect any user deferral of monitoring and complete the initial review regardless.
  - Prefer supported event registration, then a supported scheduler. Once registered, yield to the environment's wake-up mechanism rather than maintaining a polling loop.
  - Fetch current request state on wake-up. If no monitor can be registered, report that follow-up requires another invocation.
- **Coverage tracking:** Record registration IDs, owners, covered event types, and gaps. Monitor baseline request events—new commits, force-pushes or rebases, author replies, regular comments and edits, and discussion resolution-state changes (resolved/reopened)—plus each selected role's event requirements. If event coverage is incomplete, use a supported scheduled check for the gap or report the uncovered events.
- **Event handling and lifecycle:**
  - Run the initial review immediately on invocation without waiting for a quiet period. During active monitoring, group subsequent push bursts with a short quiet period (around ten minutes by default), arranging another wake-up when needed.
  - Route author explanations and reopened threads to the concern's assigned reviewers and new code to affected instances. After a force-push or rebase, re-verify any prior commit-tied fix checks against the new head.
  - When new commits introduce risks not covered by the current team, add an eligible reviewer from the effective roster (or propose it first when roster approval is required or the role is outside the effective roster), staying within the cap unless the user raises it.
  - Avoid treating the agent's own replies or resolution actions as fresh triggers.
  - Stop relevant monitors when the request closes, merges, or the user ends the assignment.
