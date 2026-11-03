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

- `schedule:weekly` (Monday before 4am), no dashboard. `prHourlyLimit` and
  `prConcurrentLimit` cap the volume. Non-major and major updates automerge once
  checks pass. Vulnerability fixes bypass the schedule.
- A 7-day `minimumReleaseAge`, so a compromised release has time to be yanked
  before it is proposed. Vulnerability fixes bypass this.
- `semanticCommits` with everything defaulting to `chore`. `config:recommended`
  maps `depType: dependencies` to `fix`, which in a release-on-commit-type
  repository publishes an artifact — so releasing is opt-in.
- Two groups: `non-major dependencies` (minor, patch, digest, pin) and
  `major dependencies`. A failing non-major bump holds the whole group until
  fixed or ignored.
- Each dependency jumps straight to its latest major instead of stepping
  through every intermediate one.
- ZirekHQ, hominux and compress4j bumps automerge and group into one
  `first-party artifacts` PR and skip the seven-day `minimumReleaseAge`. Both open a PR and Renovate
  merges it through the API (`platformAutomerge: false`) once checks pass.
  The Renovate app must be a ruleset bypass actor on any repo that requires
  reviews, or its PRs wait for a human approval. `automergeType: branch`
  (direct push, no PR) was tried first but stalls forever on any repo whose
  base branch requires PR reviews or `pull_request`-triggered status checks,
  since those checks never fire without a PR to attach to.
- `lockFileMaintenance` runs in the weekly window and automerges once checks pass, so
  transitive/lockfile versions don't rot between direct-dependency bumps.
  Renovate's vulnerability alerts only cover extracted (declared) packages, so
  this is the only path that moves a vulnerable transitive crate.
- The `pre-commit` manager is on, so hook `rev`s in `.pre-commit-config.yaml`
  get bumped like any other dependency. Renovate leaves it off by default.
- `pip_requirements` matches any `requirements*.txt`, including multi-part names
  like `requirements-test-e2e.txt` that the default pattern skips.
- The `ghcr.io/zizmorcore/zizmor` image uses `minimumReleaseAgeBehaviour:
  timestamp-optional`. Docker tags carry no release timestamp, so the 7-day
  age check would otherwise hold its bumps as pending forever.
  `zizmorcore/zizmor-action` is grouped with that image as `zizmor` and uses
  `minimumReleaseAge: 0 days`, so the action that knows a zizmor version
  arrives with it.

## What it deliberately leaves out

Runtime dependency lists and anything that re-enables `fix`. Those must be
enumerated exactly, in the repository that publishes, because a stray pattern
match ships a release.
