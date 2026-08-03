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

- Weekly schedule, `dependencyDashboard`, no automerge.
- A 7-day `minimumReleaseAge`, so a compromised release has time to be yanked
  before it is proposed. Vulnerability fixes bypass this.
- `semanticCommits` with everything defaulting to `chore`. `config:recommended`
  maps `depType: dependencies` to `fix`, which in a release-on-commit-type
  repository publishes an artifact — so releasing is opt-in.
- Build tooling, plugins and test-only dependencies grouped and kept at `chore`.

## What it deliberately leaves out

Runtime dependency lists and anything that re-enables `fix`. Those must be
enumerated exactly, in the repository that publishes, because a stray pattern
match ships a release.
