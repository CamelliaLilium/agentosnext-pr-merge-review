# Review publication protocol

Use this reference only after the read-only review is complete and the user
asks what will be sent or confirms publication.

## Confirmation card

Present a compact, exact card before any Forgejo write:

```text
Repository: agentos/AgentOSNext
PR: #<number>
Current head: <full SHA>
Current state: <open|merged|closed>
Proposed verdict: <APPROVE|REQUEST_CHANGES|HOLD>
Proposed Forgejo write: <APPROVE|REQUEST_CHANGES|comment-only|none>

Top-level body:
<exact body>

Inline comments:
- <path>:<line> [P<0-3>] <exact body>

Publish as: current authenticated Forgejo user
```

The user must confirm the PR, complete 40-hex head SHA, review state, and
payload. If the user requests an edit, regenerate the complete card and ask
again. Do not interpret an approval of the analysis as approval to publish it.

## Allowed publication forms

- `APPROVE`: publish the complete final review with the approved state only if
  the review’s own decision gate passed.
- `REQUEST_CHANGES`: publish all blocking P0–P2 findings and their evidence;
  do not omit a finding merely to make the status acceptable.
- `comment-only`: publish an observation without changing approval state. Use
  this for non-blocking P3 notes or a requested factual comment.
- `HOLD`: publish no review state. A comment explaining the hold is allowed only
  after a separate confirmation whose card says `comment-only` and includes
  the exact hold explanation.
- Inline comments: preserve stable file/line anchors. If a line moved or the
  Forgejo API cannot bind the location safely, convert it to a top-level note
  and ask for confirmation of the changed payload.

## Preflight and readback

Immediately before writing, re-fetch the PR and verify:

- the full head SHA is unchanged;
- the PR is still the same repository and number;
- the proposed state is still appropriate after new comments/reviews;
- the authenticated identity is the user’s intended Forgejo account.

State mutation is limited to an open PR. For a merged or closed retrospective
audit, `APPROVE`/`REQUEST_CHANGES` is an internal audit verdict only; Forgejo
publication is comment-only and requires explicit confirmation of that form.

After writing, re-fetch and record the resulting review/comment ID, author,
state, body checksum or exact text, head context, and timestamp. A successful
HTTP response without readback is not publication proof. A changed head,
ambiguous identity, failed request, or mismatched body cancels the operation;
do not retry automatically.

## Scope boundary

This protocol covers PR Review/Comment/Approve/Request Changes writes. It does
not cover `git push`, branch creation, merge, closing/reopening a PR, or editing
PR metadata. Those operations require a separate user request and a separate
confirmation of their target and side effects.
