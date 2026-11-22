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

[`default.json`](default.json) is the source of truth for every setting. How and when updates are created, rebased and merged — the schedule, cooldowns, major approval, automerge and lockfile maintenance — is recorded with its reasons in [ADR 0001](docs/adr/0001-dependency-update-merge-model.md). This section covers the rest.

### extends

- [config:recommended](https://docs.renovatebot.com/presets-config/#configrecommended)
- [:configMigration](https://docs.renovatebot.com/presets-default/#configmigration) — open PRs that automatically migrate deprecated config
- [:semanticCommitTypeAll(chore)](https://docs.renovatebot.com/presets-default/#semanticcommittypeallarg0) — use `chore(deps)` as the commit prefix
- [group:allNonMajor](https://docs.renovatebot.com/presets-group/#groupallnonmajor) — group minor/patch updates into a single PR
- [group:allDigest](https://docs.renovatebot.com/presets-group/#groupalldigest) — group digest updates (SHA refresh) into a single PR

### Other settings

- `prHourlyLimit: 0` — no hourly limit on how many PRs Renovate may open. The number of PRs open at the same time is still capped by `prConcurrentLimit`, which is left at Renovate's default because this preset does not set it
- All packages use `rangeStrategy: replace`: an existing version range is replaced with a new range only when the current one no longer allows the new version, instead of being widened or bumped every time
- GitHub Actions not under `actions/` are pinned to their SHA256 digest. The first PR for such an action is a `pinDigest` update, which is automerged; later SHA refreshes come through as grouped `digest` updates

## Rules for consumer repositories

### Override `platformAutomerge` when required checks are not enforced

GitHub merges without waiting for CI when no required status checks are enforced. A repository that cannot enforce them must set `"platformAutomerge": false` in its own config before enabling "Allow auto-merge".

### Keep `packageManager` free of a hash

Write the `packageManager` field in `package.json` as a plain version, with no `+sha512...` (or `+sha256...`) suffix:

```json
{
  "packageManager": "pnpm@11.19.0"
}
```

A hash makes the update PR fail with `renovate/artifacts: FAILURE` (see [ADR 0001](docs/adr/0001-dependency-update-merge-model.md)). Dropping the hash is safe with pnpm 10.34.2 or newer, which verifies the signature of the package manager it downloads.

Running `corepack use <pm>@<version>` locally writes the hash back, so check for it before committing.
