---
name: pr-implementor-babysitter
description: Follow a pull or merge request after creating it, respond to review comments, and fix accepted feedback. Use when acting as the change author, not the reviewer.
---

# PR implementor babysitter

Start after creating the PR or MR. Set up a monitor for its review comments and requested changes until it closes, merges, or the user ends the assignment. The skill defines what to do when invoked; it does not keep an agent running. Use the available host and scheduler capabilities: prefer an event trigger for review activity, then a scheduled check if event triggers are unavailable. If neither is available, tell the user that follow-up requires a new invocation. On each wake-up, fetch the current request state and act only on new, relevant feedback.

Keep a durable per-request checkpoint with processed comment and review IDs and the last handled head commit. If a reliable checkpoint is unavailable, reconstruct what has already been handled from the request history. Ignore the agent's own comments so a reply does not trigger another reply.

Read each new comment or thread in context. For valid feedback, make the smallest sound fix, run relevant checks, push the change, and reply with what changed and the commit when useful. Answer questions directly. If feedback is mistaken, explain the evidence politely; do not change correct code merely to close a thread. Avoid multiple pushes and replies for comments that can be handled together.

You may resolve a review thread yourself only when you applied the reviewer's suggested diff in full, line for line, and it covers the entire concern. For any other fix, partial application, alternative approach, or explanation without a code change, reply in the thread and leave the resolution decision to the reviewer.

Use the thread for routine clarifications, requests for more detail, and minor disagreements. Bring a consequential or stalled disagreement to the user with the reviewer's point, the relevant code or behavior, the tradeoff, and a recommendation. Examples include conflicting requirements, a scope change, or a disputed design decision. Continue with independent feedback while waiting. Do not silently dismiss or resolve the disputed thread.

Label every comment the agent posts. Use `Agent-authored` when the agent decided on or substantially wrote the point. Use `Agent-assisted at the user's direction` when the user directed the point and asked the agent to phrase or post it. This distinction depends on who directed the substance, even when the same account posts both. Include the agent name or identity when available; never imply a human wrote an agent comment.

Do not merge or close the request unless separately asked. When it closes or merges, stop its monitor and report the final status of addressed and open feedback.
