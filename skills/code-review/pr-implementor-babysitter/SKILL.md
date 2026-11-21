---
name: pr-implementor-babysitter
description: Follow code reviews of authored changes, respond to review comments, and fix accepted feedback. Use when acting as the change author, not the reviewer.
---

# Implementor babysitter

## Monitoring and wake-up

- **Setup:** Start after creating the review request on the host. Set up a monitor for review activity, checks, and branch updates until the request closes, merges, or the user ends the assignment.
- **Capabilities:** The skill defines what to do when invoked; it does not keep an agent running. Use available host and scheduler capabilities: prefer an event trigger for review and check activity, then a scheduled check if event triggers are unavailable. If neither is available, tell the user that follow-up requires a new invocation.
- **Full wake-up and status inspection:** On every wake-up—even one triggered by a check or push event—and before answering status questions or reporting merge readiness, fetch and inspect the full current request state rather than relying on cached checkpoint state:
  - All unresolved review threads, reviews, and general comments (from humans, agents, and automated reviewers),
  - CI and check status,
  - Mergeability, merge conflicts, and base-branch drift.
  Act on all new, relevant items; never let check notifications cause you to overlook open review threads.
- **Checkpoints and self-action tracking:** Keep a durable per-request checkpoint with processed comment and review IDs, the last handled head commit, and the agent's own posted comment and resolution IDs (so unlabeled agent actions on a shared host account are not mistaken for reviewer activity). If a reliable checkpoint is unavailable, reconstruct visible state from host history and re-confirm any private or unrecorded state before acting. Ignore the agent's own comments and resolution actions as fresh triggers.

## Evaluating feedback, checks, and fixes

- **Verify review claims:** Read each new comment or thread in context and check its claim against the code, intended behavior, and relevant repository boundaries.
- **Separate observation from proposed remedy:** A real problem may need a different fix when the reviewer's suggested one would introduce a wrong dependency, violate an API contract, or move responsibility into the wrong layer. Explain any different approach in the thread with the evidence behind it.
- **Answer directly and protect correct code:** Answer questions directly. If the concern itself is mistaken, explain the evidence politely; do not change correct code merely to close a thread.
- **CI failures and branch drift:** Inspect failing check logs, distinguishing code or test failures from external CI or environment failures. Fix real code failures and resolve merge or rebase conflicts cleanly, re-running relevant local checks after any rebase or base-branch update. For external failures, report the non-code cause factually (and rerun if appropriate) rather than claiming checks passed.
- **Scope discipline and no silent fallbacks:** Make the smallest sound fix scoped to the request's stated purpose—do not bundle unrelated features or unrequested API redesigns into review-fix commits. If a dependency constraint, technical blocker, or review fix prevents delivering an agreed requirement, do not silently drop or substitute it; surface the constraint, tradeoff, and proposed fallback clearly to the user (and on the request) before treating the request as ready.
- **Consolidated updates:** Run relevant local checks, push consolidated changes, and avoid multiple pushes and replies for comments that can be handled together.

## Thread resolution and escalation

- **Self-resolution rule:** You may resolve a review thread yourself only when you applied the reviewer's suggested diff in full, line for line, and it covers the entire concern, or when the reviewer explicitly marked the thread as a non-blocking advisory left to your discretion (reply briefly with whether you adopted or declined it). For any other fix, partial application, alternative approach, or explanation without a code change, reply in the thread and leave the resolution decision to the reviewer.
- **Escalation:** Use the thread for routine clarifications, requests for more detail, and minor disagreements. Bring a consequential or stalled disagreement to the user with the reviewer's point, the relevant code or behavior, the tradeoff, and a recommendation—for example, conflicting requirements, a scope change, or a disputed design decision. Continue handling independent feedback while waiting. Do not silently dismiss or resolve the disputed thread.

## Completion, reporting, and merge readiness

- **Fresh review after every push:** Track feedback and completion signals from every reviewer, including humans and agents. Treat the current head as reviewed only after all requested or participating reviewers have finished reviewing that head and their actionable threads are resolved. After you push any new commit, wait for them to review the new head again, even if they had approved or said they had no further comments on the previous one. Do not treat an earlier completion signal as covering later changes.
- **Clear pass/fail reporting:** When reporting check, build, test, or push outcomes, lead with a clear pass/fail verdict and separate non-blocking warnings from actual failures.
- **Merge readiness and policy gates:** Once all requested or participating reviewers have signed off on the current head, actionable threads are resolved, checks pass, and no agreed requirement or fallback remains unresolved, report ready-to-merge status clearly—naming the head commit, open/draft state, checks, and reviewer sign-offs. If only a host policy gate remains (such as required external approval on a shared-account repository, or draft status), call out that exact blocker so the user can approve, mark ready, or authorize a bypass; never merge or bypass branch protection unasked.

## Agent disclosure and boundaries

- **Agent disclosure:** Lead every agent-posted host comment, reply, or review with a brief note callout or admonition (using the host's callout syntax, or a plain leading blockquote when unsupported):
  - **Agent-authored** *(agent decided on or substantially wrote the point)*: `**Agent-authored** by {agent/model identity}.`
  - **Agent-assisted at the user's direction** *(user directed the substance and asked the agent to phrase or post it)*: `**Agent-assisted** by {agent/model identity} at the user's direction (on behalf of {user handle, if known}).`
  - This distinction depends on who directed the substance, even when posted from a shared account. Never imply a human wrote an agent comment.
- **Host boundaries and shutdown:** Do not merge or close the request unless separately asked. When the request closes, merges, or the user ends the assignment, stop its monitor and report the final status of addressed and open feedback; if the environment cannot stop the monitor automatically, report that explicitly so the user can clean it up.
