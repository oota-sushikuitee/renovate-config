# Common Renovate configuration for oota-sushikuitee

## About

Shared Renovate configuration to enable across the whole GitHub organization.

## Usage

```json
{
  "extends": [
    "github>oota-sushikuitee/renovate-config"
  ]
}
```

## Configuration

[`default.json`](default.json) is the source of truth for every setting. This section explains only what the file cannot say about itself: what the presets do and why the less obvious values are set.

### extends

- [config:recommended](https://docs.renovatebot.com/presets-config/#configrecommended)
- [:configMigration](https://docs.renovatebot.com/presets-default/#configmigration) — open PRs that automatically migrate deprecated config
- [:semanticCommitTypeAll(chore)](https://docs.renovatebot.com/presets-default/#semanticcommittypeallarg0) — use `chore(deps)` as the commit prefix
- [group:allNonMajor](https://docs.renovatebot.com/presets-group/#groupallnonmajor) — group minor/patch updates into a single PR
- [group:allDigest](https://docs.renovatebot.com/presets-group/#groupalldigest) — group digest updates (SHA refresh) into a single PR

### schedule / merge

- Automerge has no fixed time window; updates merge as soon as CI is green and the release-age requirement is met
- `platformAutomerge: true` — in a repository with "Allow auto-merge" enabled, GitHub merges the PR the moment its required checks pass; elsewhere Renovate falls back to merging the PR itself on its next run. GitHub merges without waiting for CI when no required status checks are enforced, so a repository that cannot enforce them must override `platformAutomerge: false` before enabling "Allow auto-merge"
- `prHourlyLimit: 0` — no hourly limit on how many PRs Renovate may open. The number of PRs open at the same time is still capped by `prConcurrentLimit`, which is left at Renovate's default because this preset does not set it
- `updateNotScheduled: true` — the schedule gates branch and PR *creation* only. A new update PR still appears only in the scheduled window, but an already open branch can be updated, rebased and automerged on any Renovate run once its cooldown has passed and CI is green, instead of waiting for the next window. Keep this value set explicitly: with `false`, Renovate returns early for out-of-window branches and never reaches the automerge step at all, so green PRs sit open until the next window. The trade-off is that an open branch may rebase and re-run CI outside the window
- `internalChecksFilter: "strict"` — an update that has not yet cleared its release-age cooldown is left out of the PR entirely instead of being shown as pending. A grouped PR can therefore list fewer dependencies than you expect; the rest arrive in a later window
- `rebaseWhen: auto` — Renovate picks the rebase behaviour per branch: branches that automerge are kept up to date with the base branch, other branches are only rebased on conflict. It used to be `conflicted`, which left grouped automerge PRs stale and orphaned when the base branch moved on, so `auto` is the value to keep

### packageRules

- All packages use `rangeStrategy: replace`: an existing version range is replaced with a new range only when the current one no longer allows the new version, instead of being widened or bumped every time
- Minor and patch updates merge unattended, so their `minimumReleaseAge` is longer than the one for majors: with no human review, a longer cooldown gives a compromised release more time to be caught and pulled
- Major updates use `dependencyDashboardApproval`: no PR is opened until someone approves it from the Dependency Dashboard, so each one is reviewed individually
- GitHub Actions not under `actions/` are pinned to their SHA256 digest. The first PR for such an action is a `pinDigest` update, which is automerged; later SHA refreshes come through as grouped `digest` updates

## Rules for consumer repositories

### Keep `packageManager` free of a hash

Write the `packageManager` field in `package.json` as a plain version, with no `+sha512...` (or `+sha256...`) suffix:

```json
{
  "packageManager": "pnpm@11.19.0"
}
```

Renovate runs `corepack use` during its npm artifact step only when the current value already contains a hash, and that command fails inside Renovate's sandbox. The result is a `renovate/artifacts: FAILURE` status on the update PR, with the hash silently dropped from the diff (see [renovatebot/renovate#37772](https://github.com/renovatebot/renovate/issues/37772)). A hash-free value skips that step entirely. Dropping the hash is safe with pnpm 10.34.2 or newer, which verifies the signature of the package manager it downloads.

Running `corepack use <pm>@<version>` locally writes the hash back, so check for it before committing. This preset cannot enforce the rule: Renovate has no configuration option that suppresses the hash.
