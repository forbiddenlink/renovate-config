# Renovate Config

Shared Renovate configuration for all forbiddenlink repos.

## Usage

Add to any repo's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>forbiddenlink/renovate-config"]
}
```

## Before enabling Renovate on a repo

Every transitive-CVE fix across this fleet is a `pnpm.overrides` entry, and Renovate can
destroy those silently.

In May 2026 `pnpm@latest` moved to 11.x and Renovate regenerated `pnpm-lock.yaml` files with
the `overrides` and `patchedDependencies` blocks **stripped from the head of the file**,
restoring vulnerable versions (`uuid@8.3.2` among them) with no signal until a human noticed
by hand ([renovatebot/renovate#43161](https://github.com/renovatebot/renovate/discussions/43161)).
The cause is Renovate reading `packageManager` only from the lockfile's sibling
`package.json` and otherwise falling back to the newest pnpm.

Two prerequisites, both of which must be true for a repo before Renovate runs against it:

1. **`packageManager` is pinned** in that repo's `package.json`. This is the actual
   mitigation, because it stops the fallback to `pnpm@latest`.
2. **The overrides guard is installed** (`.github/workflows/verify-overrides.yml`). It fails
   the build when a declared override stops being in force, which is the only thing that
   makes this class of regression visible.

`lockFileMaintenance` is **disabled** in this preset on purpose. It is the specific operation
that regenerates a lockfile wholesale, and therefore the one most likely to drop an overrides
block. Turn it on per-repo, deliberately, once both prerequisites above hold. It is genuinely
useful (it bumps transitives whose parents allow it, which Dependabot never attempts), just
not safe by default here.

## What Renovate does NOT do

It does **not** write `pnpm.overrides` for a transitive CVE. That is an
[open feature request](https://github.com/renovatebot/renovate/discussions/22049), not shipped
behaviour. Where a parent package pins the range, an override remains the only fix, and those
are maintained by hand. Renovate is a complement to that work, not a replacement for it.

## What it does

All of the following is defined in `default.json`.

- Runs on a monthly schedule: `before 5am on the first day of the month` (top-level `schedule`). Repos listed under `matchRepositories` get a weekly schedule instead (`before 5am on monday`); edit that list as projects become active or go quiet
- Groups non-major updates, so a repo gets one "non-major (automerge)" PR per scheduled run rather than one PR per package
- Waits 3 days after a release before proposing it (`minimumReleaseAge`), except for pins and digest re-pins, which skip the age gate
- Auto-merges patch updates, dev-dependency minor updates, GitHub Action updates, package-manager minor and patch updates, and pin or digest updates. Production-dependency minors are grouped as "prod minor" and need a human
- Never auto-merges `better-auth`, `stripe`, or `@biomejs/biome`
- Requires approval through the Dependency Dashboard for major updates
- Pins GitHub Action digests
- Ignores `pnpm.overrides`, `overrides`, and `pnpm-workspace.overrides` entries, so Renovate never edits the hand-maintained CVE overrides
- Limits Renovate to 3 open PRs at a time and 2 new PRs per hour, and rebases only when a PR conflicts
- Acts on vulnerability alerts immediately, including OSV, at any time of day and with no release-age wait; the schedule above does not apply to security updates

## Files

- `default.json`: the shared preset that repos extend
- `org-inherited-config.json`: organization-level config that extends the preset through `local>forbiddenlink/renovate-config`
- `.github/workflows/validate.yml`: runs `renovate-config-validator --strict` on both files for every pull request and push to `main`

## License

MIT. See [LICENSE](LICENSE).
