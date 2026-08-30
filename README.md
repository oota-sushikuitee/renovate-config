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

### extends

- [config:recommended](https://docs.renovatebot.com/presets-config/#configrecommended)
- [:configMigration](https://docs.renovatebot.com/presets-default/#configmigration) — open PRs that automatically migrate deprecated config
- [:semanticCommitTypeAll(chore)](https://docs.renovatebot.com/presets-default/#semanticcommittypeallarg0) — use `chore(deps)` as the commit prefix
- [group:allNonMajor](https://docs.renovatebot.com/presets-group/#groupallnonmajor) — group minor/patch updates into a single PR
- [group:allDigest](https://docs.renovatebot.com/presets-group/#groupalldigest) — group digest updates (SHA refresh) into a single PR

### schedule / merge

- Timezone: Asia/Tokyo
- PRs are created weekly, on Monday between 01:00 and 03:00
- Automerge has no fixed time window; updates merge as soon as CI is green and the release-age requirement is met (the repo "Allow auto-merge" setting is unavailable on the free plan, so `platformAutomerge: false` and Renovate merges directly)
- `prHourlyLimit: 0` — no hourly limit on how many PRs Renovate may open. The number of PRs open at the same time is still capped by `prConcurrentLimit`, which is left at Renovate's default because this preset does not set it
- `updateNotScheduled: false` — an already open branch is only refreshed inside the Monday window, so a PR opened on Monday is not rebased or updated again until the next window
- `internalChecksFilter: "strict"` — an update that has not yet cleared its release-age cooldown is left out of the PR entirely instead of being shown as pending. A grouped PR can therefore list fewer dependencies than you expect; the rest arrive in a later window
- `rebaseWhen: auto` — Renovate picks the rebase behaviour per branch: branches that automerge are kept up to date with the base branch, other branches are only rebased on conflict. It used to be `conflicted`, which left grouped automerge PRs stale and orphaned when the base branch moved on, so `auto` is the value to keep

### packageRules

- All packages use `rangeStrategy: replace`: an existing version range is replaced with a new range only when the current one no longer allows the new version, instead of being widened or bumped every time
- Add labels per update type (major, minor, patch, pin, pinDigest)
- Enable automerge for minor, patch, pin, pinDigest, and digest updates
- Automerged updates (minor / patch) wait 14 days after release (a longer supply-chain cooldown since they merge unattended)
- Major updates use `dependencyDashboardApproval`: no PR is opened until approved from the Dependency Dashboard (reviewed one by one); they also wait 5 days after release
- GitHub Actions not under `actions/` are pinned to their SHA256 digest. The first PR for such an action is a `pinDigest` update, which is labelled `renovate/pin-digest` and automerged; later SHA refreshes come through as grouped `digest` updates

### other

- Dependency Dashboard enabled

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
