# dependabot-sandbox

Throwaway repo to observe Dependabot behaviour before changing the real ones.

## Probes

| Entry | File | Expected |
| --- | --- | --- |
| `gomod` | `go.mod` | PR bumps `github.com/google/uuid` 1.5.0 -> 1.6.0. `go 1.26.10` unchanged. |
| `docker` | `infrastructure/Dockerfile` | PR bumps `alpine:3.20` (positive control). |
| `docker` | `infrastructure/build.yml` | **No PR** — arbitrary name, not a Dockerfile, not k8s YAML. |
| `docker-compose` | `infrastructure/compose.yaml` | PR bumps `golang:1.26.10` (proves name-based recognition). |

If `build.yml` ever shows up in a `docker` PR, the "skipped" conclusion is wrong.

## Run

1. `git init -b main && git add -A && git commit -m "test: dependabot probes"`
2. Create a private repo and push.
3. Repo -> Insights -> Dependency graph -> Dependabot -> **Check for updates**.
4. Compare PRs against the table above.
