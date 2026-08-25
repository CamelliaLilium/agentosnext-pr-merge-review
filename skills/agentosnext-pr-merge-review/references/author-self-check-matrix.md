# Author self-check matrix for reviewer audit

This matrix converts the AgentOSNext development/self-check rules into review
questions. The PR author is responsible for running the checks. The reviewer
inspects the changed files, test diffs, PR evidence, and CI/cloud job coverage;
the reviewer does not run these commands.

## Universal author obligations

- The PR is based on the latest intended `origin/dev`, uses an isolated task
  branch/worktree, and was started from a clean status. The reviewer checks the
  recorded base/head and changed scope; do not ask the reviewer to rebase it.
- Every PR has author evidence for `git diff --check`.
- The author’s final self-review covers scope, missing documentation, recovery,
  concurrency, permissions, migration risk, executable modes, realistic
  fixtures, test bypasses, and a targeted check for every new behavior.
- A bugfix includes a regression check that reproduces the pre-fix failure and
  turns green after the fix, or an exact existing check with equivalent proof.
- No test is deleted, skipped, weakened, or made fail-open to obtain green.
- Local checks are preferred and scoped. The author does not default to full
  `make check`, all-Docker, kind, full-browser, full-repository race, or a
  complete cluster deployment. Required cloud jobs are evidence when local
  prerequisites are unavailable; aggregate `required` is not a global merge
  gate.

## Sensitive-surface union

Review all rows that match the changed files; one PR may require the union of
several rows.

| Changed surface | Author’s minimum evidence to look for | Approved cloud fallback |
| --- | --- | --- |
| All PRs | `git diff --check` | none |
| `docs/**` | `make docs-lint` | `contracts` documentation contracts |
| OpenAPI, Proto, Schema, generated bindings | `make contract-check` | `contracts` |
| Ordinary Go business code | `go test -race ./<affected packages>/...` | `repo-build` + matching `go-race` shard |
| Go entrypoints/cross-package wiring | focused tests + `make go-build` | `repo-build` |
| Persistence, SQL, migration | affected persistence tests; PostgreSQL when SQL/migration semantics require it | matching `foundations` shard |
| Web | `cd web && npm test`; component changes add `npm run test:component`; type/build changes add `npm run build` | `delivery` + `web-spec-list` |
| BFF/Node runtime | focused Node tests and build | `delivery` |
| Forgejo workflows | `make forgejo-workflow-check` | `contracts` |
| Small `deploy/**` scripts | `bash -n` changed scripts + matching `*-test.sh` | `delivery` |
| Kustomize, topology, RBAC, NetworkPolicy | `make deploy-check` | `delivery` |
| Sandbox | focused scripts; topology changes also `make deploy-check` | `delivery` + `sandbox-acceptance` |
| Identity/Keycloak | matching `*-test.sh`; manifest changes also `make deploy-check` | `identity-keycloak` |
| Policy, authorization, security boundaries | affected Go race tests + policy-focused tests | matching `go-race` / `foundations` |
| Fixtures | `make fixture-test` or focused fixture test | `repo-build` |

The reviewer must not require an unrelated heavyweight command merely because
it appears in the matrix. Ask whether the changed surface is actually covered,
whether the author ran the smallest relevant check, and whether the named
cloud job is the repository-approved replacement.

## Deployment/CD audit rules

Treat any change to `.forgejo/workflows/**`, `deploy/**`, `Dockerfile*`,
deployment targets in `Makefile`, migrations, image name/tag/digest,
Secret/RBAC/NetworkPolicy, or writer Lease/flock as deployment-sensitive.

Check the PR evidence and diff for all applicable rules:

- Shell changes have syntax and focused tests; newly added or directly invoked
  shell files have Git mode `100755`.
- `cd.yml` changes include the workflow contract check; Kustomize or manifest
  changes include the deployment check.
- Writer/Lease/flock changes test the real parent-plus-child call combination,
  not only the child script.
- Kubernetes parsing tests consume real Kustomize output, not a convenient
  hand-written YAML approximation.
- Docker/kind/full-cluster work is used only when container, database, or
  cluster behavior actually changed.

An absent or contradictory required author/CI record is a reason for
`REQUEST_CHANGES`: state the exact missing evidence and changed sensitive
surface. When the diff also demonstrates an uncovered failure mode (for
example an unexecutable directly invoked script, a parent lock deadlock, or a
parser that only works on hand-written YAML), report that concrete defect as
the primary reason. Do not run the missing check yourself.
