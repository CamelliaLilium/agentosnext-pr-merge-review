---
name: agentosnext-pr-merge-review
description: Review AgentOSNext pull requests on the self-hosted Forgejo instance, especially the PR1266–PR1290 review campaign, and decide whether the exact current head is safe to merge. Use for PR URLs, PR numbers, or requests to audit open or already-merged AgentOSNext PRs. Finish read-only first; after a separate confirmation of the exact review payload, optionally publish the review state/comment as the authenticated Forgejo user. Do not code-push, merge, or mutate anything else.
metadata:
  short-description: AgentOSNext exact-head Forgejo merge review
---

# AgentOSNext PR merge review

Produce an evidence-backed review of the exact current Forgejo head. This is a
single read-only review workflow that combines the useful parts of generic PR
review, contract/state-machine review, and closure-oriented factory review. It
uses the supplied `pr-review-expert`, Apache `pr_review`, and Mastra
`factory-review` ideas as references, but it does not inherit their
DataFusion/Factory scope or their write-side effects.

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
  evidence path. Missing evidence without a demonstrated defect is normally
  `HOLD`; a reproducible uncovered defect is `REQUEST_CHANGES`.
- Never delete tests, weaken assertions, skip checks, or suggest a fail-open
  workaround as a way to make the PR mergeable.

## Safety and decision contract

- Treat the Forgejo PR page and the checked-out repository as the sources of
  evidence. Use `tea` for Forgejo PR metadata, reviews, comments, CI and review
  state; use read-only `git` inspection and the PR’s recorded command output
  for local evidence. On Windows, prefer PowerShell-compatible commands and do
  not assume Bash tools exist.
- Default to read-only. Never submit an inline comment, `APPROVE`, `REQUEST
  CHANGES`, merge, push, create a branch, or edit a PR merely because the
  review recommends it. Do those only after an explicit user instruction.
- Bind every conclusion to the exact `headSha` reviewed. A review attached to
  an older head is stale evidence, not approval of the current head.
- If a Forgejo review record does not expose the head it reviewed, treat that
  binding as uncertain until the body, commit context, or a local comparison
  establishes it; do not automatically use it to close a current finding.
- For an open PR, emit one final decision: `APPROVE`, `REQUEST_CHANGES`, or
  `HOLD`. For a retrospective audit of a merged PR, use the same vocabulary as
  an audit verdict: `APPROVE` means no surviving finding, `REQUEST_CHANGES`
  means a defect escaped into the merged result, and `HOLD` means the evidence
  cannot establish either verdict.
- `REQUEST_CHANGES` requires at least one reproducible P0/P1/P2 finding, a
  current-tree merge/conflict defect, or a required contract/evidence failure.
  `HOLD` is for missing or stale evidence, unresolved mergeability, absent
  required approval, an unverified external dependency, or a result that
  cannot be safely classified. Do not use CI redness alone as a finding: in
  AgentOSNext required CI is a quality signal, while the merge gate is approval
  based. Do report a CI/test failure as a finding when it exposes a real PR
  defect or leaves a required path unverified.

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

### 2. Comments-first and version control

Before inspecting only the patch, collect:

- all issue comments and inline review comments;
- all review records, including `REQUEST_REVIEW`, `REQUEST_CHANGES`, and
  `APPROVED`, with reviewer, timestamp, body, and head SHA if present;
- unresolved threads and whether a later response actually closes the claim;
- CI/check status at the current head and the PR's mergeability.

Reconcile each technical finding against the current tree. Mark old findings
as stale only when the changed code, exact head, or merge-tree proves the claim
no longer applies. A later approval does not erase an unresolved finding by
assertion; reproduce the prior counterexample or verify the stated closure.
If the PR has no formal technical review, say so explicitly. For an open PR,
missing required approval normally yields `HOLD`, not an inferred approval.

Before finalizing, fetch the PR metadata and review/thread list one more time.
If the head SHA, mergeability, review state, or relevant comments changed
during the review, discard the old verdict and re-review the new head (or
return `HOLD` with the drift explicitly recorded).

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

Use the campaign patterns in
[references/forgejo-review-patterns.md](references/forgejo-review-patterns.md)
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

Every P0–P2 finding must include: priority, exact file and line (or stable
 symbol), triggering input/state, observed behavior, expected frozen behavior,
 consequence, and the smallest credible fix plus a regression test. Include a
 short “actively checked and not a finding” section for tempting false
 positives. Keep CI, environment limitations, merge conflicts, and cross-PR
 reminders in separate subsections so they are not accidentally counted as
 code findings.

The final report should start with the decision and head SHA, then list
findings in priority order, evidence/verification, stale-finding closure,
cross-PR interactions, and the exact conditions for changing `HOLD` or
`REQUEST_CHANGES`. Do not claim a cloud, real-platform, database, browser, or
production test ran unless its exact evidence is available.

`APPROVE` is allowed only when the final exact head is stable, its materialized
merge is safe, required evidence and approval/threads are present, and no P0–P2
finding remains (P3 items may be listed as non-blocking). Priority wins over
evidence gaps: a reproduced P0–P2 is `REQUEST_CHANGES` even if an approval
exists; with no finding but incomplete evidence, approval, or mergeability,
use `HOLD`.

## Review publication protocol

Review publication is a separate second phase. A request to “review”, “judge
whether it can merge”, or “prepare the Codex/Claude review” authorizes only the
read-only phase. Even if the user asks for publication in the initial prompt,
finish the review and ask for confirmation of the final payload first.

After the read-only report, stop and present a publication proposal containing:

- repository and PR number;
- final exact head SHA and current PR state;
- proposed verdict: `APPROVE`, `REQUEST_CHANGES`, or `HOLD`;
- proposed Forgejo write: matching review state, comment-only, or none;
- the complete top-level review body;
- every inline comment with file, line, priority, and body, if any;
- a note that the operation will be sent as the authenticated user’s Forgejo
  identity.

Wait for an explicit confirmation that identifies the PR, accepts the state and
text, and includes or unambiguously accepts the complete 40-hex head SHA from
the proposal, for example: “确认将以上内容以 REQUEST_CHANGES 发布到 #1290，
head `<40-hex-head-sha>`”。“看起来可以”“继续”“帮我处理一下”“当前 head”
or the original review request is not sufficient confirmation. Do not silently
shorten, soften, or rewrite the confirmed payload.

If the verdict is `HOLD`, publish no Review state. A comment-only explanation
requires a new confirmation that explicitly accepts comment-only and its exact
body; never convert `HOLD` to `APPROVE` or `REQUEST_CHANGES` implicitly.

When confirmation arrives:

1. Re-read the PR metadata, current head, mergeability, unresolved threads,
   and current user identity. If the head or relevant review context changed,
   do not write; report the drift and prepare a new proposal.
2. Submit only the confirmed review/comment/status through the authenticated
   Forgejo `tea`/API write operation. `APPROVE` and `REQUEST_CHANGES` are review
   state mutations allowed for an open PR; a merged or closed retrospective
   audit may publish only an explicitly confirmed comment-only record. Never
   expose credentials.
3. Read the PR back and verify the new review/comment, state, author identity,
   head binding, and timestamp. If submission fails or the readback is
   ambiguous, stop and report it; do not retry or use another write path
   without a new confirmation.

This protocol authorizes posting the review result to Forgejo as the user. It
does not authorize a Git code push, branch creation, merge, or PR edit. A code
push is a separate mutation workflow requiring its own explicit scope and
confirmation. For detailed payload and readback rules, read
[references/review-publication-protocol.md](references/review-publication-protocol.md).
