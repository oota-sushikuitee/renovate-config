# 1. Dependency update merge model

Date: 2026-11-22

## Status

Accepted

Recorded retrospectively. The decisions below were made in the pull requests cited here; this record collects them in one place. The values live in [`default.json`](../../default.json) and are not repeated here.

## Context

This preset is shared by every repository in the organization, so its merge behaviour sets how dependency updates land everywhere. The forces, as recorded in this repository's history:

- **Updates did not merge.** The preset declared `automerge` from the start, yet nothing auto-merged and PRs piled up. Renovate's native platform automerge needs GitHub's "Allow auto-merge", which the private repositories on the current plan could not use, and they had no required status checks either. Renovate's own merge waits for CI to pass, so it is gated by CI even without required checks (#26).
- **Some repositories can use native auto-merge.** Repositories with "Allow auto-merge" and required checks can merge the moment CI passes instead of waiting for the next Renovate run. GitHub merges without waiting for CI when no required checks are enforced, so native auto-merge is only safe where checks are required (#62).
- **CI and rebase churn.** One PR per dependency meant a full CI run per PR, and every merge left the other PRs behind the base branch, triggering rebases and more CI runs (#28). A fixed automerge window could push a merge into the following week when CI finished late (#28).
- **Grouped PRs froze.** Rebasing only on conflict left grouped automerge PRs stale: the stability status stayed pending and automerge never fired, for three weeks in one consumer (#34). With `internalChecksFilter: "none"`, each new release in a group restarted the release-age clock, so grouped PRs never converged (#43).
- **Green PRs waited for the next window.** With `updateNotScheduled: false`, Renovate returns early for branches outside the schedule and never reaches its automerge step, so a green PR could sit open until the next window (#54).
- **Unattended merges carry supply-chain risk.** Minor and patch updates merge without review, so a fresh broken or malicious release would ship unseen; Renovate's [upgrade best practices](https://docs.renovatebot.com/upgrade-best-practices/) recommend a release-age wait for automerged dependencies. Majors may contain breaking changes and are reviewed one at a time (#29).
- **Lockfile-only dependencies went stale.** `rangeStrategy: replace` only moves manifest ranges, so vulnerable transitive dependencies held only in the lockfile needed manual sweep PRs (#61).
- **Hashed `packageManager` pins broke update PRs.** With a `+sha512` suffix, Renovate runs `corepack use`, which fails in its sandbox and leaves a `renovate/artifacts: FAILURE` status ([renovatebot/renovate#37772](https://github.com/renovatebot/renovate/issues/37772); #32). Renovate has no option that suppresses the hash (#54).
- The reason for a weekly window, and for its particular day and time, is not recorded: the schedule has been in place since the preset was first added.

## Decision

We will create new update branches and PRs only inside one weekly schedule window, and set `updateNotScheduled: true` so that an open branch can still be rebased and automerged on any Renovate run.

We will set `rebaseWhen: auto`, so branches that automerge are kept current with the base branch and other branches are rebased only on conflict.

We will hold every update for a `minimumReleaseAge` cooldown, longer for the unattended minor and patch updates than for majors, select versions with `internalChecksFilter: "strict"` so a group carries only versions that have cleared the cooldown, and open major updates only after approval on the Dependency Dashboard.

We will set `platformAutomerge: true`, so GitHub merges a PR as soon as required checks pass where "Allow auto-merge" is enabled, and Renovate merges it itself elsewhere. A repository that enables "Allow auto-merge" without being able to enforce required checks overrides this to `false`.

We will run `lockFileMaintenance` weekly with `automerge: true`.

We will keep `packageManager` pins free of a hash, in this repository and in every consumer.

## Consequences

Positive:

- Minor, patch, pin, digest and lockfile updates reach the default branch without human action once CI is green and the cooldown has passed.
- In-range fixes for transitive dependencies arrive through the weekly lockfile refresh instead of manual sweeps.
- Grouping plus strict version selection keeps the weekly PR count low and lets grouped PRs converge.
- Each major gets an individual, deliberate decision.

Negative:

- An open branch may rebase and re-run CI outside the schedule window.
- Keeping automerge branches current means more rebases and CI runs than rebasing only on conflict.
- Non-security fixes arrive only after the cooldown (vulnerability remediations bypass it, #43), and a grouped PR may list fewer dependencies than expected while some are still cooling down.
- A consumer that turns on "Allow auto-merge" without required checks merges untested PRs unless it overrides `platformAutomerge`; the preset cannot detect this.
- The hash-free `packageManager` rule cannot be enforced by the preset; running `corepack use` locally writes the hash back.

Neutral:

- Majors stay invisible until someone opens the Dependency Dashboard.
- Whether a repository merges natively or through Renovate depends on its own "Allow auto-merge" setting, not on this preset.
