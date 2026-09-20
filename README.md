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
  dashboard. Patch, minor and major updates automerge once checks pass, as do
  GitHub Actions digest/pin bumps.
- A 7-day `minimumReleaseAge`, so a compromised release has time to be yanked
  before it is proposed. Vulnerability fixes bypass this.
- `semanticCommits` with everything defaulting to `chore`. `config:recommended`
  maps `depType: dependencies` to `fix`, which in a release-on-commit-type
  repository publishes an artifact — so releasing is opt-in.
- Build tooling, plugins and test-only dependencies get their own groups for
  minor/patch bumps; their majors land in `major dependencies`.
- Every other update grouped by type: `patch dependencies`, `minor dependencies`
  and `major dependencies`, so a breaking minor or major bump doesn't hold up
  safe patches.
- Each dependency jumps straight to its latest major instead of stepping
  through every intermediate one.
- GitHub Actions digest/pin bumps automerge for every repo (hash-only, no new
  code). ZirekHQ bumps automerge too. Both open a PR and Renovate
  merges it through the API (`platformAutomerge: false`) once checks pass.
  The Renovate app must be a ruleset bypass actor on any repo that requires
  reviews, or its PRs wait for a human approval. `automergeType: branch`
  (direct push, no PR) was tried first but stalls forever on any repo whose
  base branch requires PR reviews or `pull_request`-triggered status checks,
  since those checks never fire without a PR to attach to.
- Monthly `lockFileMaintenance` that automerges once checks pass, so
  transitive/lockfile versions don't rot between direct-dependency bumps.
- The `pre-commit` manager is on, so hook `rev`s in `.pre-commit-config.yaml`
  get bumped like any other dependency. Renovate leaves it off by default.
- `pip_requirements` matches any `requirements*.txt`, including multi-part names
  like `requirements-test-e2e.txt` that the default pattern skips.
- The `ghcr.io/zizmorcore/zizmor` image uses `minimumReleaseAgeBehaviour:
  timestamp-optional`. Docker tags carry no release timestamp, so the 7-day
  age check would otherwise hold its bumps as pending forever.

## What it deliberately leaves out

Runtime dependency lists and anything that re-enables `fix`. Those must be
enumerated exactly, in the repository that publishes, because a stray pattern
match ships a release.
