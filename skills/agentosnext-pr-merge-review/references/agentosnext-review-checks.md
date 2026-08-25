# AgentOSNext changed-surface review checks

This reference routes an independent Reviewer to repository and contract risks
that recur in AgentOSNext. It is not sourced from, and must not be combined
with, any current or historical Forgejo Review. Always inspect the exact
current head and prove each finding from repository-owned evidence.

## Independent method

1. Bind the verdict to the full exact current head SHA and re-review after a
   head change.
2. Reconstruct the owning diff, product/architecture authority, materialized
   merge tree and changed-sensitive-surface evidence.
3. Count findings as `P0/P1/P2/P3`; keep CI quality signals separate from code
   severity.
4. For each blocking issue, cite an exact file/line or stable symbol, concrete
   triggering input/state, consequence, smallest repair and regression test.
5. Recompute closure from repository facts: migration continuity, generated
   descriptors, merge trees, owning blobs, lockfiles, executable modes,
   digests and public acceptance evidence.
6. Actively test the reasoning against plausible non-blockers such as dormant
   replicas, future activation ownership, or a failure already present on the
   exact base.

Use two complementary lenses: seek a high-signal counterexample and minimal
fix, then verify positive closure, cross-document consistency, boundary math
and whether the issue truly blocks merge.

## Changed-surface routing map

| PR/surface | Main review surface | Adversarial checks to route first |
| --- | --- | --- |
| 1266, 1290 / E12–E16 contract | IM contract and amendment | complete snapshot pagination/capacity, lease-owner claim, JS-safe integer boundaries, typed Gateway/Runtime arms, descriptor/generated parity, revision preimage and outbound provenance |
| 1273–1275 / platform drivers | Feishu/WeCom/DingTalk production drivers | real lockfile SDK dependency and export shape, platform wire limits, resolved-vs-thrown error codes, credential rotation, single-flight redemption, backpressure, retry/recovery provenance, dormant activation ownership |
| 1276–1278 / Runtime credential path | Runtime peer identity, redemption, revision restart | exact SPIFFE identity and EKU, canonical Begin/Commit/Fail reachability, read-claimed lease fencing, same-revision routing metadata, caller-owned ticket zeroing, stale generation/epoch and stop-before-start |
| 1280 / identity projection | Tombstoned Human projection | every admission consumer, enabled+tombstoned negatives, direct/group resolution, cursor progression, ledger/DB status consistency and defect-ID ownership |
| 1285 / persistence | Credential fence/migration closure | contiguous embedded migrations, real production binding path versus hand-seeded fixtures, application/personal isolation, raw error-code assertions and concurrent writer invariant |
| 1286 / destructive resources | `rm -r` contract | exact target kind, file-versus-directory semantics, observation/currentness fence, automation deregistration before delete, descendant limits, idempotent `removed` projection and missing-path guidance |
| 1288–1289 / product and acceptance docs | Semantic/acceptance freeze | accepted-vs-proposed ADR state, excluded-items boundary, complete state model, authorization predicate, public black-box evidence, exact-candidate runner wiring and no direct-DB substitution |
| Deployment/CI and merge-only maintenance | Deployment and merge ownership | immutable image/digest readback, script executable-mode closure, workflow contracts, namespace/classifier rollout interaction and materialized merge ownership |

## Recurring failure shapes

### Typed contract, incomplete state machine

Look for a new purpose, arm or enum that can be written but cannot reach the
canonical Begin/Commit/Fail path, or can replay after expiry. A test helper
that marks a row consumed directly is not evidence for the real Server path.
Require submitted tests or controlled-runner evidence for success, failure,
replay/expiry, wrong-installation and concurrent-winner paths.

### Boundary enforced locally but not on the next wire

Check page capacity, 64-bit values crossing JavaScript, platform idempotency
limits and generated descriptor parity. Require evidence on both sides of each
boundary and adjacent overflow/underflow values; inspect the downstream vendor
or Runtime wire, not only the local validator.

### Dormant does not excuse a broken owned artifact

`replicas=0` or a future activation slice can bound severity, but does not make
a PR-owned SDK adapter, contract consumer, production factory or current-state
README internally correct.

### Tests bypass the failing authority

Red flags include hand-seeded owners, fake SDKs where production dependencies
are claimed, direct SQL replacing a public audit journey, regex scans matching
comments rather than a target SQL constraint, or positive fixtures that always
provide an optional field. Require a submitted negative fixture at the real
production boundary; do not create or execute it as the Reviewer.

### Merge-only heads need ownership proof

Compare the subject parent, exact current base and current tree. Prove PR-owned
blobs and modes are unchanged and that conflict resolution did not drop an
existing workflow gate or security check. Do not re-litigate unrelated dev-side
changes; report a real intersection with owning paths.

## Evidence wording

Prefer concrete statements:

- “Current head `<sha>` is bound; the exact migration tree has versions 1..117
  without a gap, and submitted continuity evidence covers that invariant.”
- “The fixture passes only after hand-seeding the owner; production Create
  still leaves the row undiscoverable, so this is P1.”
- “CI has one failure also present on the exact base; it is a separate quality
  signal and not counted as a new P2.”

Avoid “looks good”, “CI is green”, or “a later PR will fix it” without exact
repository evidence and an ownership boundary.
