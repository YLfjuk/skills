# Execution and review ownership

## Resolve capabilities

Inspect the current environment's supported model discovery, child delegation, conversation launch, host access, and monitoring capabilities. Resolve every reviewer's model and policy before launch. Instances of the same role may use the same or different providers and models. Use documented interfaces and lifecycle rules rather than assuming particular tool names or a harness.

Use child delegation for `subagents` and separate top-level conversations for `threads`. Select the configured provider and model through a mechanism that supports them. If the requested model or execution mode is unavailable, explain the limitation and apply the model selection policy or ask which execution alternative to use. Do not silently switch modes.

Use the configured host connection for request state and discussions. Register the review URL with the current workspace or conversation if the environment supports or requires it. Report registration failures.

## Reviewer brief

Include concrete values, the selected role's full definition and procedures, and instance-specific settings in every brief. Read any metadata requirements declared by the role, collect the requested values using its defaults and inference rules, and include them in the brief. Ask for required values that cannot be established reliably; independent reviewers may proceed. Reviewers must not need access to the orchestrator's personal role file.

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
Available relevant skills: {resolved skill names and locations}
Apply pr-review-babysitter when available, subject to the ownership below.
Discover and apply other available skills relevant to this scope and changed code.
Posting mode: {host posting authorized, or read-only}
Ownership: {mode-specific ownership text below}
Return the head actually reviewed, findings with evidence and useful locations,
overlapping discussions, unresolved concerns, verification limits,
and whether this assigned scope is complete.
Do not approve, merge, close, or implement changes to this request.
Use an isolated checkout for verification that could disturb another workspace.
```

For independent second opinions, give each instance the same source evidence and let it reach its initial judgment before sharing sibling findings. Reviewers may inspect existing host discussions. Compare results after their initial assessments; independent review does not authorize duplicate host comments.

## Subagents

Include this ownership text:

> Return findings and assessments to the orchestrator. Do not post host comments, resolve host discussions, or register monitoring. The orchestrator owns validation, deduplication, posting, replies and resolution of its findings, and monitoring. This overrides posting and monitoring defaults in any applied reviewer skill. Finish the review round after returning your result.

Retain execution IDs, instance ID, selected model, role, and reviewed head in the checkpoint. Use a stable retry identifier per instance and round where supported. After a failed or ambiguous launch, inspect existing work before retrying. Avoid starting a second equivalent task for an instance whose round is active; intentional parallel instances of one role are separate work.

On new commits or author responses, re-engage affected instances using the delegation interface's supported lifecycle. Include the original brief, exact new head, prior findings, author responses, and unresolved objections. If a new task is required, retain its ID as a new round. Use returned execution handles as documented; backing storage or conversation identifiers are not automatically continuation targets. If the head changes during a round, mark coverage stale and review the latest head before claiming current completion.

## Threads

The user must select `threads` to authorize separate top-level reviewer conversations. Include this ownership text:

> You own posting and follow-up for your assigned findings. Check existing discussions and coordinate overlaps with the orchestrator before posting a concern another reviewer owns. Report findings, host IDs, reviewed head, unresolved concerns, completion, and monitor registration status. Respect read-only mode when configured.

Create one conversation per reviewer instance, including its role and instance ID in the title and its full brief as the initial prompt. Configure provider, model, and intended workspace through the supported launch interface. Verify inherited workspace settings when the interface lacks a selector. Prompt instructions alone do not establish a conversation's workspace binding.

Retain conversation IDs keyed by instance. Confirm readiness when launch preparation is asynchronous. Inspect existing conversations after ambiguous launch results before retrying. Another instance of the same role is not a duplicate. Continue follow-up through the interface's supported conversation lifecycle with the updated head and prior review context.

Maintain overall status without posting a second copy of reviewers' findings. Assign one owner to overlapping concerns. Register coordination monitoring only when needed and give it a distinct purpose from reviewers' finding follow-up.

## Findings and resolution

Validate findings against the cited commit, requirements, and current discussion history. Consolidate duplicates with each other and existing feedback, retaining contributing instance IDs and evidence. Post one actionable discussion per concern using a resolvable mechanism when available. If the host lacks one, track resolution in the checkpoint. Agreement between reviewers does not replace evidence. Preserve substantive disagreement and explain the decision.

Label agent-authored comments as such. Verify the substance of a fix or explanation before marking a concern addressed. Resolve only discussions owned by the configured posting owner unless otherwise authorized. Do not treat a host's resolved status as proof of a fix. Scope completion summaries to finished instances and named commits.

## Monitoring

Respect any user deferral of monitoring and complete the initial review regardless. Prefer supported event registration, then a supported scheduler. Once registered, yield to the environment's wake-up mechanism rather than maintaining a polling loop. Fetch current request state on wake-up. If no monitor can be registered, report that follow-up requires another invocation.

Record registration IDs, owners, covered event types, and gaps. Include each selected role's event requirements. If event coverage is incomplete, use a supported scheduled check for the gap or report the uncovered events.

Group bursts of pushes with a short quiet period, around ten minutes by default, arranging another wake-up when needed. Route author explanations to the concern's assigned reviewers and new code to affected instances. Propose additional coverage when risks change, staying within the cap unless the user raises it. Avoid treating the agent's own replies as fresh triggers. Stop relevant monitors when the request closes, merges, or the user ends the assignment.
