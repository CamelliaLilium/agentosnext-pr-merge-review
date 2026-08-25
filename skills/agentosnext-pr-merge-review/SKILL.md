---
name: agentosnext-pr-merge-review
description: Independently review an open AgentOSNext pull request on the self-hosted Forgejo instance at its exact current head. Act as a peer Reviewer alongside Claude Reviewer and Codex Reviewer without reading or relying on their reviews. Decide only APPROVE or REQUEST_CHANGES, explain the reasons, show the exact review body to the user, and publish it as the authenticated Forgejo user only after separate full-SHA confirmation. Do not code-push, merge, or mutate anything else.
metadata:
  short-description: AgentOSNext exact-head Forgejo merge review
---

# AgentOSNext PR merge review

Produce an evidence-backed independent review of the exact current Forgejo
head. The authenticated user acts as a peer Reviewer at the same workflow
level as Claude Reviewer and Codex Reviewer; do not impersonate either bot or
synthesize their verdicts. The only outcomes are `APPROVE` and
`REQUEST_CHANGES`, each with a concise reasoned review body.

## Reviewer boundary

This Skill evaluates someone else’s PR. The PR author’s “start development”,
local self-check, and final self-review duties are review criteria, not duties
for the reviewer to perform.

- Do not create a worktree or branch, edit the checkout, implement a fix, or
  prepare a follow-up PR.
- Do not run tests, builds, `make check`, `make docs-lint`, `make contract-check`,
  race suites, Docker, kind, browser suites, deployment scripts, or cloud jobs.
  Inspect the PR’s test changes, command claims, CI artifacts, and coverage;
  report whether they are sufficient. Read-only Forgejo metadata, exact diff,
  object, and merge-tree inspection is allowed.
- Do not turn a missing local command into an invitation to run it. First check
  whether the corresponding controlled/cloud job is the repository-approved
  evidence path. If evidence required to establish merge safety is missing,
  use `REQUEST_CHANGES` and state exactly what must be supplied.
- Never delete tests, weaken assertions, skip checks, or suggest a fail-open
  workaround as a way to make the PR mergeable.

## Safety and decision contract

- Treat the Forgejo PR page and the checked-out repository as the sources of
  evidence. Use `tea` only for PR metadata, exact base/head, changed files,
  mergeability and CI/check status during the read-only phase; use read-only
  `git` inspection and the PR's recorded command output for local evidence. Do
  not call Review/comment/thread listing endpoints. On Windows, prefer
  PowerShell-compatible commands and do not assume Bash tools exist.
- Default to read-only. Never submit an inline comment, `APPROVE`, `REQUEST
  CHANGES`, merge, push, create a branch, or edit a PR merely because the
  review recommends it. Do those only after an explicit user instruction.
- Bind every conclusion to the exact `headSha` inspected. A proposal prepared
  for an older head is invalid for the current head.
- Do not retrieve, quote, compare, reconcile, or rely on other reviewers'
  Review bodies, states, findings, approvals, requests, or unresolved threads.
  They are independent peer outputs and are outside this Review's evidence.
- For an open PR, emit exactly one final decision: `APPROVE` or
  `REQUEST_CHANGES`.
- `REQUEST_CHANGES` requires at least one reproducible P0/P1/P2 finding, a
  current-tree merge/conflict defect, or required contract/test evidence that
  the author has not supplied. Do not use CI redness alone as a finding: in
  AgentOSNext required CI is a quality signal, not the global merge gate. Do
  report a CI/test failure when it exposes a real PR defect or leaves a changed
  sensitive surface without the evidence required to approve it.

## Review workflow

### 1. Establish the review target

Resolve the PR URL or number to `agentos/AgentOSNext`, record the PR state,
base branch and SHA, head branch and SHA, draft status, mergeability, author,
updated time, and changed-file summary. For a range such as PR1266–PR1290,
make a small ledger with one row per PR; do not silently review only the last
open PR.

Read the repository `AGENTS.md` and `CLAUDE.md` (they must be byte-identical),
then navigate the documentation map through `docs/README.md` and the relevant
L1 indexes. Read only the product, architecture, decision, testing, defect,
deployment, or contract leaves touched by the PR. Code is current-state
authority; `docs/` is target-state authority; a plan or PR body is not proof.

Audit, rather than perform, the author’s start-of-development discipline:
latest `origin/dev` base, task branch/worktree isolation, clean status, and
the declared sensitive surfaces. A moving branch name or an unbound base is a
review finding; the reviewer does not repair it by fetching or rebasing.

### 2. Independent evidence and head control

Collect only evidence owned by the PR and repository:

- PR metadata, body, base/head SHAs, changed files and mergeability;
- the exact patch and materialized merge tree;
- repository product/architecture/contract authorities;
- submitted test changes and author-recorded local evidence;
- current-head CI/check results as independent quality evidence.

Do not open the PR's Review list, review comments, requested-review records, or
unresolved review threads. Do not mention whether Claude Reviewer, Codex
Reviewer, a Human Reviewer, or any other Reviewer approved or rejected it.

Before finalizing, fetch the PR metadata one more time. If the head SHA,
mergeability, PR state, or changed-file set moved during the review, discard
the old verdict and review the new head again. If the PR is no longer open,
stop without preparing a Forgejo Review-state write.

### 3. Freeze the candidate and inspect the actual delta

Use the PR base/head SHAs, not a moving branch name. Inspect the exact diff and
the author/CI evidence for the mandatory `git diff --check`; do not run the
check or any other test as a substitute. Rebuild the materialized merge with
the current base (`git merge-tree` or an equivalent non-mutating inspection)
and distinguish:

- the PR's owning diff;
- mechanical changes brought in by a merge from `dev`;
- an actual conflict resolution that rewrote the PR's owning behavior.

For a PR whose head is a merge commit, compare the prior reviewed head with the
current tree and verify owning blobs, modes, generated files, and digests. A
textually clean merge is not proof of semantic compatibility.

### 4. Classify the change and choose adversarial checks

Classify the PR as docs-only, API/proto/contract, Go runtime, TypeScript/web,
database/migration, deployment/CI, SDK/driver, or a cross-cutting combination.
Take the union of all applicable sensitive surfaces and reason through the
smallest counterexamples that could make the claimed behavior false. Inspect
the submitted tests and recorded evidence; do not execute the counterexamples
yourself:

- state machines: every transition, replay, expiry, cancellation, stale
  generation, and concurrent winner;
- contracts: both positive arms and missing, duplicate, cross-arm, boundary,
  overflow, and wire-compatibility negatives;
- persistence: contiguous migration versions, real transaction/error paths,
  unique fences, CAS predicates, DB-time, and rollback/forward-only semantics;
- security/identity: exact principal, owner, epoch, lease and fail-closed
  checks before lookup or mutation; no second authority or sensitive projection;
- drivers/SDKs: a lockfile-instantiable production adapter and real export or
  wire shape, not only a fake injected fixture; retry, backpressure, shutdown,
  and reconnect behavior;
- resource/destructive actions: observed target kind, confirmation/currentness,
  idempotent `removed` semantics, child side effects, and “stop before delete”;
- docs/tests/acceptance: target-state and current-state alignment, public
  evidence versus direct database evidence, exact candidate binding, and a
  runnable cloud/controlled-runner entry rather than a prose-only plan.

For a bugfix, require a submitted regression check that would fail on the
pre-fix behavior and pass on the fix, or identify the precise existing check
that provides the same proof. Reject test deletion, skipped coverage, narrowed
assertions, and fail-open changes as review findings even when the remaining
suite is green.

Read the author’s self-review diff summary and status evidence as a checklist:
scope containment, code-to-documentation alignment, recovery/concurrency/
permission/migration risks, executable modes, realistic fixtures, absence of
test bypasses, and a targeted test for each new behavior. These are evidence
to verify, not extra commands for the reviewer to run.

For the author-check expectation by sensitive surface, read
[references/author-self-check-matrix.md](references/author-self-check-matrix.md).
Use it to audit whether the PR supplied the right local/cloud evidence and
whether its tests cover the changed surface; never execute the matrix as part
of this review.

### 5. Cross-PR and closure review

Check shared contract files, migrations, generated bindings, docs ledgers,
lockfiles, and deployment manifests against adjacent open and merged PRs in the
campaign. A patch can be individually valid yet unsafe when a neighboring PR
uses a different revision preimage, integer encoding, ticket state, owner
fence, SDK boundary, or acceptance claim. Report the interaction and evidence;
do not invent an ordering recommendation unless the repository or user has
explicitly authorized one.

This is a code/contract interaction check only. Do not read or report any
Review attached to those neighboring PRs.

Use the changed-surface checks in
[references/agentosnext-review-checks.md](references/agentosnext-review-checks.md)
when the PR is in or adjacent to PR1266–PR1290. The patterns are routing
signals and observed examples, not pre-decided findings: re-check every item
against the current head.

### 6. Synthesize and report

Use severity P0–P3:

- P0: catastrophic security/data-loss or an immediately unsafe production
  path;
- P1: merge-blocking boot failure, broken core flow, authorization bypass,
  duplicate side effect, or durable state corruption;
- P2: important correctness/contract/consumer/evidence gap that should block a
  clean merge but has a bounded impact or dormant activation path;
- P3: non-blocking robustness, documentation, test-quality, or maintainability
  issue.

Every blocking finding must include: priority, exact file and line (or stable
symbol), triggering input/state, observed behavior, expected frozen behavior,
consequence, and the smallest credible fix plus a regression test. Keep CI and
environment limitations separate from code findings.

The user-facing result is a publication proposal, not a synthesis of other
reviews. It contains only the repository/PR, exact head SHA, proposed Forgejo
state, and the complete reasoned Review body. The body should be concise but
specific: decision, P0/P1/P2/P3 counts when useful, blocking reasons for
`REQUEST_CHANGES`, or the main safety/coverage reasons for `APPROVE`. Do not
claim a cloud, real-platform, database, browser, or production test ran unless
its exact evidence is available.

`APPROVE` is allowed only when the final exact head is stable, its materialized
merge is safe, required author/CI evidence is present, and no P0–P2 finding
remains (P3 items may be noted as non-blocking). Otherwise use
`REQUEST_CHANGES` and give the exact defect or missing required evidence.

## Review publication protocol

Review publication is a separate second phase. A request to “review”, “judge
whether it can merge”, or “prepare the Codex/Claude review” authorizes only the
read-only phase. Even if the user asks for publication in the initial prompt,
finish the review and ask for confirmation of the final payload first.

After the read-only report, stop and present a publication proposal containing:

- repository and PR number;
- final exact 40-hex head SHA and confirmation that the PR is open;
- proposed Forgejo Review state: `APPROVE` or `REQUEST_CHANGES`;
- the complete top-level review body;
- a note that the operation will be sent as the authenticated user's own
  independent Reviewer identity, alongside (not as) Claude/Codex Reviewer.

Wait for an explicit confirmation that identifies the PR, accepts the state and
text, and includes or unambiguously accepts the complete 40-hex head SHA from
the proposal, for example: “确认将以上内容以 REQUEST_CHANGES 发布到 #1290，
head `<40-hex-head-sha>`”。“看起来可以”“继续”“帮我处理一下”“当前 head”
or the original review request is not sufficient confirmation. Do not silently
shorten, soften, or rewrite the confirmed payload.

When confirmation arrives:

1. Re-read only the PR metadata, exact head, PR state, mergeability, and current
   authenticated user identity. Do not read other Reviews or threads. If the
   head or PR state changed, do not write; review the new head and prepare a new
   proposal.
2. Submit exactly one top-level formal Review through authenticated Forgejo
   `tea`/API: the confirmed `APPROVE` or `REQUEST_CHANGES` state plus the
   confirmed body. Do not create inline or comment-only records. Never expose
   credentials.
3. Read back only the newly submitted Review by its returned ID and verify its
   state, author identity, exact body, head binding, and timestamp. Do not use
   other Reviews as evidence. If submission fails or readback is ambiguous,
   stop and report it; do not retry or use another write path without a new
   confirmation.

This protocol authorizes posting the review result to Forgejo as the user. It
does not authorize a Git code push, branch creation, merge, or PR edit. A code
push is a separate mutation workflow requiring its own explicit scope and
confirmation. For detailed payload and readback rules, read
[references/review-publication-protocol.md](references/review-publication-protocol.md).
