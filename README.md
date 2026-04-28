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

## What It Does

- Groups all non-major updates into one PR per week (Monday mornings)
- Auto-merges patches and trusted minor updates
- Groups related packages (Sentry, Supabase, Next.js, Prisma)
- Pins major updates for manual review
- Security vulnerability alerts always enabled
