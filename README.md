# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset used across my
repositories, so dependency policy lives in one place instead of being copied
per repo.

## Usage

In a repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>austek/renovate-config"]
}
```

Add repository-specific rules alongside the `extends`. Anything that could
publish an artifact belongs there, not here.

## What the preset sets

- No schedule (`prHourlyLimit` and `prConcurrentLimit` cap the volume), no
  dashboard. Major updates never automerge; matching minor/patch updates do,
  per the groups below.
- A 7-day `minimumReleaseAge`, so a compromised release has time to be yanked
  before it is proposed. Vulnerability fixes bypass this.
- `semanticCommits` with everything defaulting to `chore`. `config:recommended`
  maps `depType: dependencies` to `fix`, which in a release-on-commit-type
  repository publishes an artifact — so releasing is opt-in.
- Build tooling, plugins and test-only dependencies grouped and kept at `chore`.
- Every other minor/patch update grouped into one `non-major dependencies` PR;
  majors stay ungrouped so each gets individual review.
- GitHub Actions digest/pin bumps automerge for every repo (hash-only, no new
  code). ZirekHQ patch/minor bumps automerge too. Both open a PR and merge it
  automatically via GitHub's native auto-merge (`platformAutomerge`) once
  checks -- and, where required, review -- pass. `automergeType: branch`
  (direct push, no PR) was tried first but stalls forever on any repo whose
  base branch requires PR reviews or `pull_request`-triggered status checks,
  since those checks never fire without a PR to attach to.
- Monthly `lockFileMaintenance`, so transitive/lockfile versions don't rot
  between direct-dependency bumps.

## What it deliberately leaves out

Runtime dependency lists and anything that re-enables `fix`. Those must be
enumerated exactly, in the repository that publishes, because a stray pattern
match ships a release.
