<!-- markdownlint-disable -->

# Hardening Report: gacts--directory-listing/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--directory-listing/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks.

.github/workflows/demo.yml:
  - actions/checkout@v6 (line 24)
  - peaceiris/actions-gh-pages@v4 (line 33)

.github/workflows/release.yml:
  - actions/checkout@v6 (line 15)
  - gacts/github-slug@v1 (line 16)

.github/workflows/tests.yml:
  - actions/checkout@v4 (line 23)
  - gacts/gitleaks@v1 (line 25)
  - actions/checkout@v6 (line 29)
  - actions/setup-node@v4 (line 30)
  - actions/checkout@v6 (line 36)
  - actions/setup-node@v4 (line 37)
  - actions/upload-artifact@v7 (line 41)
  - actions/checkout@v6 (line 51)
  - actions/download-artifact@v8 (line 52)
  - stefanzweifel/git-auto-commit-action@v7 (line 53)
  - actions/checkout@v6 (line 60)

Locations:

- `.github/workflows/demo.yml:24`
- `.github/workflows/demo.yml:33`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:36`
- `.github/workflows/tests.yml:37`
- `.github/workflows/tests.yml:41`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:52`
- `.github/workflows/tests.yml:53`
- `.github/workflows/tests.yml:60`

### script-injection (severity: high)

Sub-rule (a): In .github/workflows/release.yml, the expression `${{ github.actor }}` is interpolated directly inside a `run:` shell block. This allows YAML template substitution to inject arbitrary shell metacharacters before the shell ever sees the value. Affected lines:
  - `git config --local user.name "${{ github.actor }}"`
  - `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
These should be moved to an `env:` block and the env var double-quoted in the shell script.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### missing-permissions (severity: medium)

Two workflow files lack a top-level `permissions:` block and have jobs without any job-level `permissions:` key, meaning those jobs run with the default (potentially broad) token permissions.

- .github/workflows/release.yml: No top-level permissions and the sole job `update-git-tag` has no job-level permissions block.
- .github/workflows/tests.yml: No top-level permissions and four of five jobs (gitleaks, eslint, dist-built, run-this-action) have no job-level permissions block.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all 15 unpinned action references to full 40-character commit SHAs with original tags preserved as comments. Actions pinned: actions/checkout (v4→34e1148, v6→df4cb1c), peaceiris/actions-gh-pages (v4→84c30a8), gacts/github-slug (v1→83cd3d9), gacts/gitleaks (v1→c9a0338), actions/setup-node (v4→49933ea), actions/upload-artifact (v7→043fb46), actions/download-artifact (v8→3e5f45b), stefanzweifel/git-auto-commit-action (v7→4a55954).

2. **script-injection**: In release.yml, moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from inline `run:` shell interpolation into the step's `env:` block as `GIT_ACTOR` and `GITHUB_TOKEN`, then referenced them as plain shell variables.

3. **missing-permissions**: Added `permissions: {}` at the top level of release.yml and tests.yml. Added job-level `permissions: contents: write` to release.yml's update-git-tag job. Added `permissions: contents: read` to tests.yml's gitleaks, eslint, dist-built, and run-this-action jobs (commit-and-push-fresh-dist already had explicit permissions).

