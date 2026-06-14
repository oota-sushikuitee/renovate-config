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
- No limit on the number of PRs created at once
- `rebaseWhen: conflicted` — rebase only on conflict, avoiding needless rebases / CI re-runs every time the base branch advances

### packageRules

- Add labels per update type (major, minor, patch, pin)
- Enable automerge for minor, patch, pin, and digest updates
- Automerged updates (minor / patch) wait 14 days after release (a longer supply-chain cooldown since they merge unattended)
- Major updates use `dependencyDashboardApproval`: no PR is opened until approved from the Dependency Dashboard (reviewed one by one); they also wait 5 days after release
- GitHub Actions not under `actions/` are pinned to their SHA256 digest

### other

- Dependency Dashboard enabled
