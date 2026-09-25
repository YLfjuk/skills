---
name: pr-review-babysitter
description: Review a pull or merge request, then follow its comments and new commits until the review is resolved. Use when acting as the ongoing general reviewer, not the change author.
---

# PR review babysitter

Review the PR or MR as a general code reviewer. Identify actionable defects with evidence from the change and relevant repository context. When posting an issue, use the host's review or discussion mechanism that lets the team track it as unresolved until addressed. Attach it to a relevant code location when that helps; use a request-level resolvable thread for an issue that has no useful single location. Do not leave an actionable finding only as a general comment that the team cannot track to resolution. General comments are fine for a review summary or an open discussion. If the host has no resolvable mechanism, state the finding clearly and track its status in the review checkpoint. Avoid repeating existing feedback or inventing issues to fill a review.

After the initial review, set up a monitor for the same PR or MR until it closes, merges, or the user ends the assignment. The skill defines what to do when invoked; it does not keep an agent running. Use the available host and scheduler capabilities: prefer an event trigger for comments and commit updates, then a scheduled check if event triggers are unavailable. If neither is available, tell the user that follow-up requires a new invocation. On each wake-up, fetch the current request state before deciding whether work is needed.

Keep a durable per-request checkpoint with the last reviewed head commit and processed comment or review IDs. If a reliable checkpoint is unavailable, reconstruct what has already been reviewed from the request history before posting. Ignore the agent's own comments as triggers for a reply.

For new commits, wait for a short quiet period after the latest push, around ten minutes by default, so a burst of pushes gets one review. If woken during that period, arrange another check after it ends and leave the head unreviewed in the checkpoint. Compare the last reviewed head with the latest head. Check whether prior findings were addressed and whether the new changes introduced defects. A response without a code change calls for a focused reply, not a full code review. If the head changes during review, refresh it before posting conclusions tied to that head.

Own the decision to resolve findings you raised. Review the implementor's change or explanation, then resolve the thread when the concern is addressed. The implementor may resolve a thread without waiting for you only when they applied your suggested diff in full, line for line, and that diff covers the entire concern. If they changed the approach, applied only part of the suggestion, or answered without changing code, leave the resolution decision to you. Treat routine questions and minor disagreements as part of the thread discussion.

Label every comment the agent posts. Use `Agent-authored` when the agent decided on or substantially wrote the point. Use `Agent-assisted at the user's direction` when the user directed the point and asked the agent to phrase or post it. This distinction depends on who directed the substance, even when the same account posts both. Include the agent name or identity when available; never imply a human wrote an agent comment.

Do not approve, merge, or close the request unless separately asked. When the request closes or merges, stop its monitor and report the final review status.
