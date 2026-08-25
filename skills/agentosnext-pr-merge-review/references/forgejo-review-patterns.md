# Forgejo review patterns for the PR1266–PR1290 campaign

This reference is distilled from the Codex/Claude review records visible on
`https://forgejo.huafucius.top/org/agentos/pulls` on 2026-08-25. It is a review
heuristic, not an inventory of defects. Always fetch the current PR and verify
the exact head before using a pattern.

## Shared method observed in both reviewer families

The strongest reviews consistently did the following:

1. Bound the verdict to a short SHA and replaced stale findings after each new
   head.
2. Read prior issue/review/inline comments before judging the patch
   (“comments-first”).
3. Counted findings as `P0/P1/P2/P3` and separated CI status from code
   severity.
4. Used exact file/line references and a concrete adversarial input or state,
   then requested the smallest repair and a regression test.
5. Rechecked closure rather than trusting a commit message: migration
   continuity, generated descriptors, materialized merge trees, owning blobs,
   lockfiles, permissions, digests, and public evidence were independently
   recomputed.
6. Added an explicit rebuttal section for plausible but non-blocking concerns
   (for example dormant replicas, a future activation owner, or a CI failure
   already present on the base).

Codex reviews tended to lead with a high-signal counterexample and a minimal
fix. Claude reviews tended to independently verify the positive closure,
cross-document consistency, exact boundary math, and whether a finding was
actually blocking. Use both lenses; resolve disagreement by reproducing the
behavior, not by voting.

## Campaign routing map

| PRs | Main review surface | Adversarial checks to route first |
| --- | --- | --- |
| 1266, 1290 | E12/E16 IM contract and amendment | complete snapshot pagination/capacity, lease-owner claim, JS-safe integer boundaries, typed Gateway/Runtime arms, descriptor/generated parity, revision preimage and outbound provenance |
| 1273–1275 | Feishu/WeCom/DingTalk production drivers | real lockfile SDK dependency and export shape, platform wire limits, resolved-vs-thrown error codes, credential rotation, single-flight redemption, backpressure, retry/recovery provenance, dormant activation ownership |
| 1276–1278 | Runtime peer identity, redemption, revision restart | exact SPIFFE identity and EKU, canonical Begin/Commit/Fail reachability, read-claimed lease fencing, same-revision routing metadata, caller-owned ticket zeroing, stale generation/epoch and stop-before-start |
| 1280 | Tombstoned Human projection | every admission consumer (E1, Knowledge, Admin, direct/group resolution), enabled+tombstoned negative cases, cursor progression, ledger/DB status consistency, defect-ID ownership |
| 1285 | Credential fence/migration closure | contiguous embedded migrations, real production binding path versus hand-seeded fixtures, application/personal isolation, raw error-code assertions, concurrent writer invariant |
| 1286 | Destructive resource `rm -r` contract | exact target kind, file-versus-directory semantics, observation/currentness fence, automation deregistration before delete, descendant limits, idempotent `removed` projection, missing-path guidance |
| 1288–1289 | Product/docs/acceptance freeze | accepted-vs-proposed ADR state, excluded-items boundary, complete state model including Human acceptance, authorization predicate, public black-box evidence, exact-candidate runner wiring, no direct-DB substitution |
| 1267, 1270, 1281–1284, 1287 | Deployment/CI and merge-only maintenance | immutable image/digest readback, script executable-mode closure, workflow count/contracts, namespace and classifier rollout interaction, materialized merge ownership; absence of review records is an evidence gap |

## Recurring failure shapes

### Contract looks typed but the state machine is incomplete

Look for a new purpose, arm, or enum that can be written but cannot reach the
canonical Begin/Commit/Fail path, or can be replayed after expiry. A unit test
that marks a row consumed directly is not evidence for the real Server path.
Require the submitted tests or controlled-runner evidence to cover a real
success, failure, replay/expiry, wrong-installation, and concurrent-winner
journey.

### Boundary math is asserted but not enforced at the next wire

Examples in this campaign included page capacity, 64-bit values crossing
JavaScript, platform idempotency-key limits, and generated descriptor parity.
Require submitted tests/evidence for both sides of each boundary and adjacent
overflow/underflow values; inspect the downstream vendor or runtime wire, not
just the local validator.

### “Dormant” does not excuse a broken owned artifact

`replicas=0`, “activation is PR8”, or a future deployment seam can justify not
raising a dormant issue to P1, but it does not make a PR-owned SDK adapter,
contract consumer, production factory, or current-state README correct. Keep
the severity bounded while still requiring the owning slice to be internally
coherent.

### Tests pass because they bypass the failing authority

Red flags are hand-seeded owners, fake SDKs where production dependencies are
claimed, direct SQL substituted for a public audit journey, regex scans that
match comments instead of the target SQL constraint, and positive fixtures
that always include an optional field. Look for a submitted mutant or negative
fixture exercising the production boundary; do not create or execute one as
the reviewer.

### Merge-only heads need a different proof

When a reviewer sees a merge commit, compare the old approved head, the exact
current base, and the current tree. Prove the PR-owned blobs and modes are
unchanged and that any conflict resolution did not drop an existing workflow
gate or security check. Do not re-litigate unrelated dev-side changes, but do
record a real intersection with the PR's owning paths.

## Expected evidence wording

Prefer statements such as:

- “Current head `…` is bound; prior `REQUEST_CHANGES` was stale because the
  exact migration tree now has versions 1..117 with no gap, and the targeted
  continuity test passes.”
- “The fixture passes only after hand-seeding the owner; the production Create
  transaction still leaves the row undiscoverable, so this remains P1.”
- “CI is 5 success/10 skipped/1 failure and the failure is also present on the
  base; it is reported separately and is not counted as P2.”

Avoid “looks good”, “CI is green”, or “the next PR will fix it” without the
corresponding exact evidence and ownership boundary.
