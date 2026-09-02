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

## Before enabling Renovate on a repo — read this

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
   mitigation — it stops the fallback to `pnpm@latest`.
2. **The overrides guard is installed** (`.github/workflows/verify-overrides.yml`). It fails
   the build when a declared override stops being in force, which is the only thing that
   makes this class of regression visible.

`lockFileMaintenance` is **disabled** in this preset on purpose. It is the specific operation
that regenerates a lockfile wholesale, and therefore the one most likely to drop an overrides
block. Turn it on per-repo, deliberately, once both prerequisites above hold — it is genuinely
useful (it bumps transitives whose parents allow it, which Dependabot never attempts), just
not safe by default here.

## What Renovate does NOT do

It does **not** write `pnpm.overrides` for a transitive CVE — that is an
[open feature request](https://github.com/renovatebot/renovate/discussions/22049), not shipped
behaviour. Where a parent package pins the range, an override remains the only fix, and those
are maintained by hand. Renovate is a complement to that work, not a replacement for it.

## What It Does

- Groups all non-major updates into one PR per week
- Auto-merges patches and trusted minor updates; prod minors and majors need a human
- Majors require approval via the Dependency Dashboard
- Pins GitHub Action digests
- Acts on vulnerability alerts immediately, including OSV
