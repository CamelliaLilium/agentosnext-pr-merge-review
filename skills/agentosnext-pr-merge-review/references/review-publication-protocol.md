# Review publication protocol

Use this reference only after the read-only review is complete and the user
asks what will be sent or confirms publication.

## Confirmation card

Present a compact, exact card before any Forgejo write:

```text
Repository: agentos/AgentOSNext
PR: #<number>
Current head: <full SHA>
Current state: open
Proposed Forgejo Review state: <APPROVE|REQUEST_CHANGES>

Top-level body:
<exact body>

Publish as: current authenticated Forgejo user's independent Reviewer identity
```

The user must confirm the PR, complete 40-hex head SHA, review state, and
payload. If the user requests an edit, regenerate the complete card and ask
again. Do not interpret an approval of the analysis as approval to publish it.

## Allowed publication forms

- `APPROVE`: publish the complete final review with the approved state only if
  the review’s own decision gate passed.
- `REQUEST_CHANGES`: publish all blocking P0–P2 findings and their evidence;
  do not omit a finding merely to make the status acceptable.

The only allowed records are formal `APPROVE` and `REQUEST_CHANGES` Reviews.
Put all reasons and file/line references in the single top-level Review body;
do not create comment-only or inline records.

## Preflight and readback

Immediately before writing, re-fetch the PR and verify:

- the full head SHA is unchanged;
- the PR is still the same repository and number;
- the PR is still open and the materialized merge context has not changed;
- the authenticated identity is the user’s intended Forgejo account.

Do not retrieve other Review bodies, states, review comments, or threads during
preflight. The current user is an independent peer Reviewer and does not need
to reconcile Claude Reviewer, Codex Reviewer, or any other Reviewer.

State mutation is limited to an open PR. If the PR is merged or closed, do not
publish anything through this Skill.

After writing, fetch the newly returned Review ID and record its author, state,
body checksum or exact text, head context, and timestamp. Do not enumerate or
interpret other Reviews. A successful
HTTP response without readback is not publication proof. A changed head,
ambiguous identity, failed request, or mismatched body cancels the operation;
do not retry automatically.

## Scope boundary

This protocol covers one formal PR Review write: `APPROVE` or
`REQUEST_CHANGES`, with its reason body. It does not cover standalone comments,
inline comments, `git push`, branch creation, merge, closing/reopening a PR, or
editing PR metadata. Those operations require a separate workflow and are not
performed by this Skill.
