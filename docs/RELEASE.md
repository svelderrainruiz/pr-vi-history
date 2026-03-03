# Release Runbook

This runbook keeps `pr-vi-history` releases deterministic and easy to consume.

## Scope

- Repository: `svelderrainruiz/pr-vi-history`
- Public surface:
  - `.github/workflows/pr-vi-history.yml`
  - `.github/workflows/vi-history-linux-compare.yml`

## Pre-release checklist

1. `main` is green for reusable workflow smoke and compare checks.
2. Hosted Linux compare run has produced report + capture artifacts.
3. README usage examples match current workflow inputs/outputs.
4. Caller integration branch has validated the target ref.

## Release steps

1. Select release version (`vX.Y.Z`) and target commit on `main`.
2. Create the GitHub release from the target commit.
3. Move major tag `v1` to the same commit.
4. Verify:
   - release page exists
   - `v1` resolves to the new commit
   - caller dispatch succeeds with the new ref

## Example commands

```powershell
$repo = 'svelderrainruiz/pr-vi-history'
$tag = 'v1.1.6'
$target = (git rev-parse origin/main).Trim()

gh release create $tag `
  --repo $repo `
  --target $target `
  --title $tag `
  --notes 'Release notes go here.'

git tag -f v1 $target
git push origin refs/tags/v1 --force

gh release view $tag --repo $repo
```

## Caller repin guidance

- Stabilization:
  - pin caller to exact release tag (for example `@v1.1.6`)
- Continuous:
  - pin caller to major tag `@v1` once release confidence is established

## Post-release evidence

Capture links in the release issue or PR:

- release URL
- one green harness/caller run URL
- artifact URL containing compare report and capture JSON
