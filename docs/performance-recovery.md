# APEX Prime performance recovery

Investigation date: October 28, 2026.

**Status: blocked by missing source. No performance result is certified.**

## Evidence

- [The initial commit](https://github.com/bdickie9/apex-prime-engine/commit/80a41b65d68afb346b324ebb9f60ab8a0a0131b0)
  has no parents and contains only `.github/workflows/perf.yml`. It references
  `ci/perf.sh` but does not include it.
- All five available branches were traced to their roots: seven distinct
  commits, six distinct file trees. The complete file-path union is
  `.github/workflows/perf.yml`, `LICENSE`, `README.md`, and `perf-trigger.txt`.
  There is no application, benchmark, dependency manifest, submodule pointer,
  or source archive in those trees. The PR #2 merge ref contains the same files
  as its head. No release was returned by the releases endpoint, and the Git
  refs listing contains no tags.
- The [existing main-branch README](https://github.com/bdickie9/apex-prime-engine/blob/4b852deb43990199ffb9185d77987152bb2476a9/README.md)
  calls this a public remnant and directs readers to MIDAS and later APEX work.
  It does not identify a commit or export containing the missing Prime app.
- A further 138 branch snapshots across ten related FLINTTECH repositories
  were inspected recursively. All returned complete trees. No snapshot contains
  `ci/perf.sh`; exact-path commit-history queries on all 138 branches also
  returned no commits for that path. Other applications and a separate
  synthetic Continuum benchmark were found, but no matching Prime harness.
- Nine distinct saved APEX source archives were inspected. None contains
  `ci/perf.sh`, the `perf-output` contract, or a source reference identifying it
  as this repository's Prime application. Performance-monitoring utilities and
  unit-test dependencies in another APEX app do not supply the missing load
  driver, workload, or acceptance criteria.
- [The original PR #2 run](https://github.com/bdickie9/apex-prime-engine/actions/runs/40629315094)
  reached the preflight and exited 1 for missing `ci/perf.sh`. AWS credential,
  benchmark, and artifact-upload steps did not execute. The run has no artifact.

These findings cover accessible references and inspected exports. They cannot
exclude an inaccessible repository, an unprovided export, or unreachable Git
objects removed from accessible history. Empty indexed code-search results
were not treated as proof of absence.

## Source required before performance testing can succeed

| Required material | What must be supplied or recovered |
| --- | --- |
| Application identity and complete source | An immutable repository/commit or complete export for the intended APEX Prime application, including its entry point, imports, and build configuration. The APEX name alone does not establish which related app is the target. |
| Original executable harness | `ci/perf.sh` and every benchmark/load-driver file it invokes, including fixtures, assertions, setup, teardown, and report generation under `perf-output/`. |
| Reproducible runtime and dependencies | The application's runtime version, dependency manifests and lock files, plus the benchmark tool and test dependencies. This repository does not establish a language or package manager. |
| Disposable service setup | Any required schema, migrations, seed data, and local test-service configuration. The workflow names `DATABASE_URL`, `REDIS_URL`, and `S3_BUCKET`; those references alone do not establish which services or schemas the real harness requires. |
| Measurement contract | The original operations/endpoints, input dataset, concurrency, warmup, duration, metrics, and any pass/fail thresholds or baseline. Recover these rather than inventing values to obtain a green check. |

If the original harness cannot be supplied, a new benchmark requires a
separately defined target and workload. Such work would be a new benchmark,
not recovery or proof of any earlier performance claim.

## Workflow behavior

PR #2 fixes unsupported direct `secrets` references in step conditions by
using job environment variables. The preflight runs before AWS authentication,
fails for a missing or empty `ci/perf.sh`, and checks shell syntax before
execution. The benchmark command still propagates failures, and artifact
upload still fails if no report exists. Artifact retention is limited to one
day when a real report eventually exists.

Keep this PR as a draft until genuine source and a runnable benchmark are
restored and reviewed. Do not add a placeholder script, skip the preflight,
mask benchmark errors, or substitute another application's test results.

The job uses the standard `ubuntu-latest` runner in this public repository.
[GitHub documents standard public-repository runner compute as free](https://docs.github.com/en/billing/concepts/product-billing/github-actions).
The missing-source check exits before AWS authentication and produces no
performance artifact. Restoring a script is not permission to provision cloud
resources: inspect its service calls and use disposable local services for
validation before executing it.

## Validation boundary

YAML parsing, embedded-shell syntax validation, and preflight control-flow
checks validate the workflow guard. GitHub accepting and executing the job
validates that its expressions parse. An expected preflight failure is useful
diagnostic evidence; it is not a successful application-performance benchmark.
