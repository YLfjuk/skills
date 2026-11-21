---
name: pr-review-babysitter
description: Perform code reviews on the review host, including a one-time review. Post findings now even if the user defers monitoring; follow later activity unless deferred. Use when acting as the general reviewer and findings belong on the review host.
---

# Code review babysitter

## Review scope and checks

- Perform a general code review of the requested changes. Identify actionable defects with evidence from the change and relevant repository context.
- Read applicable repository standards and the references needed to interpret them before concluding the review.
- Check whether added or moved code and data belong in their files and layers, even when the issue suggests a location; call out conflicts between the issue and repository conventions.
- **Centralized behavior or data:** When a change centralizes behavior or data, trace the consumers it is meant to serve. Search code and content files for remaining direct implementations or hard-coded values, and check whether those consumers use the shared source.
- **Duplicate constants and domain values:** For added or changed constants, configuration values, and variables holding fixed domain values, search beyond the diff for existing definitions of the same concept—by meaning, related names, and literal values, since different names or equivalent expressions can hide duplication.
  - Inspect matching definitions and their consumers to determine whether they encode the same rule and should change together.
  - Multiple variables or configuration entries referencing the same canonical constant are valid reuse. Independently redeclaring its value or expression for the exact same rule creates a second source of truth, even if the values currently match; flag that duplication when it can drift, citing both locations and the concrete consequence.
  - Equal values alone are not evidence: keep independently owned rules separate, and check dependency boundaries before recommending a shared import. Apply this check even when the change does not claim to centralize anything.
- **Requirements conformance:** Find the originating issue or spec through the request description, linked tracker items, or branch name. Compare the implementation with its requirements and check whether those requirements make sense in the repository.
  - Raise missing or partial behavior as a finding and quote the relevant requirement.
  - For materially unrequested behavior, explain why it exceeds the stated scope and why that matters.
  - If a requirement itself appears mistaken or conflicts with repository behavior or standards, ask the author about it and show the evidence, even when the code follows it.
  - If no originating issue or spec is available, say so in the review summary rather than inferring requirements from the implementation.

## Findings, calibration, and posting

- **Visibility and severity calibration:**
  - Before classifying a change as breaking, verify its public-contract or export visibility and check actual codebase call sites.
  - Calibrate blocking vs. non-blocking severity by impact and plausibility in the repository's operating context—separating merge-blocking defects from low-probability edge cases or speculative hardening—while preserving helpful non-blocking suggestions as resolvable threads clearly labeled as advisory/non-blocking and explicitly passing the decision to the implementor or maintainer (for example, noting that it is optional and up to them to adopt or resolve).
- **Resolvable discussions:** When posting an issue, use the host's review or discussion mechanism that lets the team track it as unresolved until addressed. Attach it to a relevant code location when that helps; use a request-level resolvable thread for an issue that has no useful single location.
  - Do not leave an actionable finding only as a general comment that the team cannot track to resolution. General comments are fine for a review summary or an open discussion.
  - If the host has no resolvable mechanism, state the finding clearly and track its status in the review checkpoint.
  - Avoid repeating existing feedback or inventing issues to fill a review.
- **Posting-mode overrides:** Respect explicit user or caller posting-mode instructions across initial and follow-up passes: when directed to run read-only or in staged/draft mode, return all drafted host mutations (findings, replies, summaries, and proposed resolutions, including agent-disclosure callouts) without mutating the host directly, and follow any monitoring instruction independently.
- **Workspace isolation:** If local verification would write files, switch branches, or otherwise disturb the implementor's workspace, use a temporary worktree at the commit being reviewed. Do not edit files or run mutating tests in a checkout the implementor may be using. Reuse an existing isolated checkout when one is available, and remove temporary worktrees after verification. Read-only inspection does not require a new worktree.

## Monitoring and checkpoints

- **Setup and deferral:** Complete the initial review immediately on invocation without waiting for a quiet period. Afterward, set up a monitor for the same review request until it closes, merges, or the user ends the assignment. If the user defers or declines monitoring, complete the review and post its findings now (unless read-only or staged), then skip the monitor.
- **Capabilities:** The skill defines what to do when invoked; it does not keep an agent running. Use available host and scheduler capabilities: prefer an event trigger for comments and commit updates, then a scheduled check if event triggers are unavailable. If neither is available, tell the user that follow-up requires a new invocation.
- **Live host refresh:** On each wake-up—and before reporting finding status, answering status questions, or posting conclusions—refresh live host state (commits, checks, discussions, and review threads). Never report cached checkpoint state as current host truth.
- **Checkpoints and self-action tracking:** Keep a durable per-request checkpoint with the last reviewed head commit, processed comment or review IDs, and the agent's own posted comment and resolution IDs (so unlabeled agent actions on a shared host account are not mistaken for human activity). If a reliable checkpoint is unavailable, reconstruct visible state from host history before posting and re-confirm any private or unrecorded state before resuming affected actions. Ignore the agent's own comments and resolution actions as fresh triggers.
- **Subsequent pushes and replies:** For new commits after the initial review, wait for a short quiet period after the latest push (around ten minutes by default) so a burst of pushes gets one review. If woken during that period, arrange another check after it ends and leave the head unreviewed in the checkpoint.
  - Compare the last reviewed head with the latest head. Check whether prior findings were addressed and whether the new changes introduced defects.
  - A response without a code change calls for a focused reply, not a full code review.
  - If the head changes during review, refresh it before posting conclusions tied to that head.

## Resolution and completion

- **Owning resolution:** Own the decision to resolve findings you raised. Verify the implementor's fix or counterargument against the actual code, named commit, and originating issue or spec before resolving or withdrawing a finding; do not treat a host "resolved" state as proof of a fix, and do not withdraw a valid finding without checking the evidence.
  - The implementor may resolve a thread without waiting for you only when they applied your suggested diff in full, line for line, and that diff covers the entire concern, or when you explicitly marked the thread as a non-blocking advisory left to their discretion. Even for advisory threads, still check the thread's status and whether any change was made on later passes. If they changed the approach, applied only part of the suggestion, or answered without changing code on a blocking finding, leave the resolution decision to you.
  - Treat routine questions and minor disagreements as part of the thread discussion; bring consequential or stalled disagreements to the user with the evidence and tradeoff.
  - When accepting a deferral or agreed exception, record the brief rationale (or follow-up reference) in the thread before resolving it.
- **Already-resolved threads on a shared account:** When a tracked thread is already marked resolved on a shared host account, verify the fix or decision in the code against a named commit, check your own action log to rule out self-resolution, do not assume the resolution was unauthorized or reopen it without cause (if the substance check shows the defect remains, reopen or follow up with the evidence), and report both the substance check and that the resolving actor cannot be distinguished on the shared account.
- **Auditing self-owned threads:** Whenever a newer review pass supersedes an earlier self-owned summary or hold discussion—and before posting a completion summary—audit all resolvable threads opened by this reviewer and resolve superseded ones with a brief note pointing to the updated head or summary so no stale self-owned thread remains open.
- **Completion summary:** Once you have no remaining findings on the current head commit, post one clear review summary so the implementor knows your review is complete.
  - Name the reviewed commit and state that you have no further comments; this is also appropriate when the initial review finds nothing.
  - Report conformance with the originating issue or spec separately from repository standards, and note any requirements you questioned.
  - Scope the statement to your own review. Check other reviewers' visible status and do not imply the whole request is ready while their feedback is open or pending.
  - Do not repeat the summary on idle checks with no new commits. A later commit makes your earlier completion statement stale: review the new head and post a fresh completion summary naming it once no remaining findings apply to it.

## Reporting, decision summary, and scorecard

- **Local work-thread updates:** After each substantive review pass, leave a brief update in the local work thread, separate from the host review discussion. State the head reviewed, what findings were opened or resolved, what remains open, and what you are waiting for. Link to review threads when useful. Summarize outcomes rather than copying the diff or discussion history; skip updates for checks that found nothing new.
- **Decision summary:** Include a decision summary in each substantive update and the final reply:
  - Link the request and name the reviewed head.
  - Refresh the live host state before reporting: open/draft/closed/merged, current head, conflicts, required checks, required approvals, and unresolved review threads.
  - Report whether it is mergeable now as `yes`, `no`, or `unknown`, with the concrete blockers or missing information.
  - Distinguish conflict-free merging, host merge requirements, non-host workflow prerequisites, and your review recommendation—never mislabel a non-host workflow step as a host merge blocker.
  - If required checks fail for non-code CI or runner reasons, report the host check failure factually while noting the non-code cause and any local verification performed.
  - If the head has changed since the last review, still report current host mergeability factually for the latest head, but mark the review assessment stale and recommend `wait` (never `merge`) until the new head is reviewed.
- **Compact scorecard:**
  - Choose criteria relevant to this code review based on its purpose, scope, and risks, honoring user-specified criteria. There is no fixed roster or required number of criteria.
  - Score each assessed criterion from 1 to 10 (higher scores mean greater readiness and fewer concerns):
    - **1:** Serious concerns
    - **5:** Mixed evidence or meaningful gaps
    - **10:** Strong evidence with no material concerns found
  - Give a brief evidence-based reason and any remaining concern for each score. For any score below 10, explicitly state whether the gap reflects open defects, bounded verification limits, or both.
  - Mark relevant criteria with insufficient evidence as `unknown` rather than inventing a score or omitting the uncertainty. Passing CI alone does not justify high scores across criteria.
- **Recommendation:** End with a recommendation to `merge`, `wait`, or `request changes`, and the reasons that matter to the user's decision.
  - Keep blocking findings and unmet host requirements explicit regardless of scores; do not average them away.
  - A recommendation does not authorize approval or merging.
  - The decision summary is informational: in live posting mode, continue posting findings and follow-up replies within the authorized scope without waiting for approval of the summary (respecting read-only or staged overrides when configured).

## Agent disclosure and boundaries

- **Agent disclosure:** Lead every agent-posted host comment, reply, or review with a brief note callout or admonition (using the host's callout syntax, or a plain leading blockquote when unsupported):
  - **Agent-authored** *(agent decided on or substantially wrote the point)*: `**Agent-authored** by {agent/model identity} ({role, if applicable}).`
  - **Agent-assisted at the user's direction** *(user directed the substance and asked the agent to phrase or post it)*: `**Agent-assisted** by {agent/model identity} at the user's direction (on behalf of {user handle, if known}).`
  - This distinction depends on who directed the substance, even when posted from a shared account. Never imply a human wrote an agent comment.
- **Host boundaries and shutdown:** Do not approve, merge, or close the request unless separately asked. When the request closes, merges, or the user ends the assignment, stop its monitor and report the final review status; if the environment cannot stop the monitor automatically, report that explicitly so the user can clean it up.
