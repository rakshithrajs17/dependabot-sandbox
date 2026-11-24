# dependabot-sandbox

Throwaway repo to observe Dependabot behaviour before changing the real repos.

## Results

| Entry | File | Observed | PR |
| --- | --- | --- | --- |
| `gomod` | `go.mod` | `uuid` 1.5.0 -> 1.6.0. `go 1.26.10` **unchanged**. | #2 |
| `docker` (control) | `infrastructure/Dockerfile` | `alpine:3.20` -> `3.24`. | #3 |
| `docker` (arbitrary name) | `infrastructure/build.yml` | **No PR.** Skipped. | — |
| `docker-compose` | `infrastructure/compose.yaml` | `golang:1.26.10` -> `1.27.4`. | #1 |
| `docker-compose` (digest) | `infrastructure/compose.golang.yaml` | tag `1.26.10` -> `1.27.4` **and** digest `87c65a68...` -> `70bd7dde...`. | #1 |
| `docker` (older alpine) | `infrastructure/Golang.Dockerfile` | Pending: `golang:1.26.10-alpine3.23`. | — |
| `docker-compose` (older alpine) | `infrastructure/compose.alpine.yaml` | Pending: `golang:1.26.10-alpine3.23`. | — |

## Conclusions so far

- `docker` reads Dockerfiles/Containerfiles plus Kubernetes/Helm YAML only. Arbitrary names like `build.yml` are skipped even though they contain `image:`.
- `docker-compose` reads by filename. `compose.yaml` and `compose.<variant>.yaml` match; `build.yml` does not.
- `gomod` updates module requirements and leaves the `go` directive/toolchain alone.
- Compose tag bumps rewrite the `@sha256:` digest to the new tag's index digest in the same PR.
- Without an ignore rule, `1.26.x` -> `1.27.x` is a semver-minor update and is proposed automatically. A `golang` `version-update:semver-minor`/`semver-major` ignore is required to keep Go majors manual.

## Docker Hub note

`golang:1.26.10-alpine*` only publishes `alpine3.23` and `alpine3.24`; there is no 3.21/3.22 for that Go patch.

## Run

1. `git add -A && git commit -m "test: dependabot probes"`
2. `git push`
3. Repo -> Insights -> Dependency graph -> Dependabot -> **Check for updates**.
4. Compare PRs against the results table.
