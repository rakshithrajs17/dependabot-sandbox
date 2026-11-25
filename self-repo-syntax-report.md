# `$/` cross-repo verification

**Date:** 2026-11-25
**Verdict:** PASS — GitHub Actions `$/` resolves to the **called** workflow's repository and commit, not the caller's.

## Why this was tested

`affordmed/action-build` is a reusable-workflow library called from service repositories. Switching its internal action references from versioned tags to `$/.github/actions/<name>` only works if `$/` inside a reusable workflow resolves to **action-build**, not to the service repository that called it. This test proves that behavior on live GitHub Actions runners.

## Setup

| Role | Repository | Ref |
| --- | --- | --- |
| Caller | `rakshithrajs17/dependabot-sandbox` | branch `test/self-repo-syntax` |
| Library | `rakshithrajs17/dependabot-sandbox-lib` | `main` @ `938e8e9` |

Both repos contain an action at `.github/actions/echo-context` with a different marker:

- caller marker: `self-repo-syntax-marker-v1`
- library marker: `lib-marker-v1`

The caller invokes the library's reusable workflow:

```yaml
cross-repo-lib:
  uses: rakshithrajs17/dependabot-sandbox-lib/.github/workflows/reusable-lib.yml@main
```

That workflow resolves its action with:

```yaml
- uses: $/.github/actions/echo-context
```

## Results

| Case | Reference | Result |
| --- | --- | --- |
| Direct action step | `$/.github/actions/echo-context` | PASS |
| Inside a composite action | `nested/action.yaml` → `$/…` | PASS |
| Reusable workflow called with `./` | `./.github/workflows/reusable.yml` | PASS |
| Reusable workflow called by full path | `owner/repo/.github/workflows/reusable.yml@ref` | PASS |
| Cross-repo reusable call | `dependabot-sandbox-lib@main` | PASS |

Runs:

- same-repo: https://github.com/rakshithrajs17/dependabot-sandbox/actions/runs/45608263748
- cross-repo: https://github.com/rakshithrajs17/dependabot-sandbox/actions/runs/45608575247

## Cross-repo evidence

Caller head: `00191b666eb42ba0d8ca24593946d2d7e6186d2a`
Library head: `938e8e95cfdf91344b4e9a8def996e0cc11d8834`

```text
caller_repository=rakshithrajs17/dependabot-sandbox
job.workflow_ref=rakshithrajs17/dependabot-sandbox-lib/.github/workflows/reusable-lib.yml@refs/heads/main
job.workflow_sha=938e8e95cfdf91344b4e9a8def996e0cc11d8834

Download action repository 'rakshithrajs17/dependabot-sandbox-lib@938e8e95...' (SHA:938e8e95...)
action_repository=rakshithrajs17/dependabot-sandbox-lib
action_ref=938e8e95cfdf91344b4e9a8def996e0cc11d8834
action_path=/home/runner/work/_actions/rakshithrajs17/dependabot-sandbox-lib/938e8e95.../.github/actions/echo-context
marker=lib-marker-v1
```

The caller had the same action path but a different marker. The action came from the library's commit and read the library's marker, so `$/` did not fall back to the caller.

## Runner support

- GitHub-hosted runner used: `2.338.2`
- Org self-hosted image: `2.337.0`
- Minimum for `$/`: `2.336.0`

`$/` is GitHub.com only (not GHES), which matches this org.

## Caveats

- Tested public↔public. `affordmed/action-build` is private; the caller token already reads it to fetch the reusable workflow, so the `$/` action fetch uses the same access.
- `actionlint` 1.7.12 does not parse `$/`. The action-build repo needs the narrow ignore in `.github/actionlint.yaml` that targets only `$/.github/actions/` refs; missing refs on other actions are still reported.
- `act` 0.2.98 cannot parse `$/`, so local act runs of converted workflows stop working. The lockfile CLI is unaffected.

## Implications for action-build

- All 12 internal action references can use `$/.github/actions/<name>`; an internal fix ships with the workflow and cannot drift behind a stale tag.
- The lockfile drops the `affordmed/action-build` self-dependency and pins the transitive actions of internal composites per workflow (for example, the Azure actions inside `deployment-tools`).
- `gh actions-lock --verify` passes with 14/14 workflows valid after the switch.

## Cleanup (not run)

```bash
git push origin --delete test/self-repo-syntax
gh repo delete rakshithrajs17/dependabot-sandbox-lib --yes
```
