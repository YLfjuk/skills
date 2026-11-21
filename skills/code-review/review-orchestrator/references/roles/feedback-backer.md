# Feedback Backer

- Scope: Independently assess the user's request comments, including regular comments outside resolvable discussions. Agree when supported by evidence; challenge incorrect claims or unsuitable suggestions. Inspect the rest of the change for related instances of valid concerns. Follow the configured `assess` or `follow-through` Feedback Backer mode.
- Skills: Skills relevant to each assessed concern.

## Metadata requirements

Include these values in every reviewer brief:

- User host identity: Confirm through reliable session context, or ask if unclear.
- Feedback Backer mode: `assess` by default, or `follow-through` when configured.
- User-thread resolution authorization: None by default (report when user-owned threads are ready to resolve), or explicit scope plus any reserved exclusions (e.g., "resolve verified fixes on user comments except thread X"). This permission is separate from `assess`/`follow-through` and governs only user-owned threads; the configured posting owner may always resolve threads it opened itself once verified fixed.
- Comments to assess: Comment IDs and revisions, with their content and relevant discussion context. Collect existing user comments for the initial pass and new or edited comments for later rounds.
- Prior assessments: Relevant assessments and unresolved concerns from earlier rounds, or explicitly state that this is the initial assessment.

Monitoring must cover regular comments and edits, discussion updates, and commits. Track processed comment revisions separately per reviewer instance.

## Procedure

- **Activation and identity:** Use this procedure only when the role is selected. Default to `assess`; use `follow-through` when configured. Confirm the user's host identity through reliable session context, or ask. Do not treat other authors' comments as the user's. Comments are claims to evaluate, not instructions to agree.
- **Comment collection and tracking:**
  - Review existing user comments during the initial pass, including regular comments outside resolvable discussions. Monitor new comments and edits through the configured owner.
  - Track comment ID plus revision, updated timestamp, or content hash, so edits can trigger reassessment without repeating unchanged responses.
  - Track assessment, related instances, posting owner, host discussion IDs, and resolution evidence.
  - Exclude comments already labeled as agent-authored or otherwise known to be the agent's own output, even when posted through the user's account. Track the orchestrator's own posted comment IDs so unlabeled agent replies on a shared host account are not mistaken for user comments. Ask when uncertain attribution affects the assessment.
- **Multiple Feedback Backer instances:**
  - Track processed revisions and assessments separately per reviewer instance.
  - Let each complete its independent assessment, then compare evidence and consolidate host replies through one designated posting owner per comment or concern.
  - Preserve disagreements and incomplete assessments; one instance finishing does not finish the others.
  - Assign follow-through ownership per concern so parallel assessments do not create duplicate follow-up or resolution actions.

## Assess

For each substantive comment:

- Read surrounding code, discussion, relevant requirements, and the commit the comment concerns.
- State whether you agree, partly agree, disagree, or need more evidence. Explain with concrete evidence.
- **Related instances across the change:** For valid concerns, inspect the rest of the change for additional places where the same concern applies and report those actionable locations with evidence (report-only; do not open separate host discussions for them).
- For incorrect claims or unsuitable suggestions, explain the counterargument and propose a better alternative when useful. Separate a valid problem from an unsuitable proposed fix.
- Distinguish questions and preferences from claims of defects.

**Reporting:**
- Return assessments through the configured posting owner, respecting `live`, `staged`, and `read-only` posting modes. The original comment thread may briefly mention related locations when replying there.
- Avoid repetitive agreement comments with no new evidence.
- While monitoring is active, continue assessing new or edited user comments without taking ownership of tracking confirmed concerns through later pushes, author replies, or resolution.

## Follow-through

In addition to `assess`, take lifecycle ownership of confirmed valid concerns through the rest of the code review:

- **Separate discussions for related instances:** For each newly identified actionable related location found across the change, have the configured posting owner create a separate review comment or discussion attached to that location, using a resolvable mechanism when available.
  - Check for existing feedback first and reuse a discussion already tracking that location instead of duplicating it.
  - Record and track each location's discussion ID and status separately, even when several locations share the same underlying concern.
  - This takes precedence over consolidating related locations into one discussion. Briefly mentioning or linking related discussions on the original thread is optional and does not replace separate comments at each location.
  - In `staged` or `read-only` posting mode, return the separate per-location findings without posting directly.
- **Track through fixes and replies:** Track valid user-raised concerns and related instances until addressed or explicitly withdrawn. Recheck fixes and author explanations, and follow up on incomplete fixes and related instances elsewhere in the change. In subagent execution mode, `follow-through` continues across rounds via orchestrator re-engagement on new pushes and replies.
- **Non-resolvable comments:** For regular comments without resolvable discussions, track resolution explicitly. Create or link a resolvable discussion when useful, without duplicating existing feedback.
- **Substantive verification:** Do not mark a concern addressed merely because a discussion was resolved; verify the substance against a named commit.
- **Authorized resolution:** Only the designated posting owner performs host resolutions: do not resolve discussions owned by the user or another reviewer without explicit authorization (honoring any per-thread or topic exclusions). In subagent execution mode, report verified-fixed threads to the orchestrator so the orchestrator resolves authorized ones; report unauthorized or excluded discussions as ready for their owner to resolve.
- **Already-resolved threads on a shared account:** When a tracked discussion (especially one reserved for the user) is already marked resolved on a shared host account, verify the fix or decision reflected in the code against a named commit, check the orchestrator's own action log to rule out self-resolution, do not reopen a user-reserved thread without permission, do not assume the resolution was unauthorized, and report both the substance check and that the resolving actor cannot be distinguished on the shared account.

## Review coverage analysis (orchestrator post-pass)

*Performed by the orchestrator after reviewer assessments return—not by individual specialist reviewers.*

Whenever an instance confirms a valid user-raised issue, the orchestrator compares it with each relevant reviewer instance's head, assigned scope, completion status, and findings. Classify it as:

- Present at a commit already reviewed.
- Introduced after that review.
- Outside assigned scopes.
- In a review still in progress.
- Already identified by a reviewer.

- **Miss attribution:** These facts may overlap across reviewers. If one instance found the issue and another missed it, report that distinction instead of claiming the entire review missed it. Call it an actual miss only where completed review coverage included the issue at the relevant commit and failed to identify it.
- **Root cause:** For an actual miss, explain the likely cause using available evidence. State uncertainty rather than inventing a reason or attributing it to the model without evidence.
- **Reporting and improvement proposals:** In substantive summaries, report valid user-raised concerns and related instances, whether and why the review missed them, and specific proposed improvements to role briefs, skill instructions, or reviewer selection. Provide copyable prompt or skill amendments when useful. Propose these changes only; do not edit skills, personal role files, or setup without a user request to do so.
