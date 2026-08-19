<!-- markdownlint-disable -->

# Hardening Report: gacts--directory-listing/v1.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--directory-listing/v1.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.actor }}` is directly interpolated inside a `run:` shell command block. YAML template substitution occurs before the shell sees the value, allowing an attacker-controlled actor name to inject shell metacharacters. Offending lines: `git config --local user.name "${{ github.actor }}"` and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`. These should be moved to an `env:` block and the env var double-quoted in the script.

Locations:

- `.github/workflows/release.yml:19`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags instead of immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the tag is moved. Failing references in demo.yml: `actions/checkout@v5`, `peaceiris/actions-gh-pages@v4`. Failing references in release.yml: `actions/checkout@v5`, `gacts/github-slug@v1`. Failing references in tests.yml: `actions/checkout@v4`, `gacts/gitleaks@v1`, `actions/checkout@v5`, `actions/setup-node@v4` (×2), `actions/upload-artifact@v5`, `actions/download-artifact@v6`, `stefanzweifel/git-auto-commit-action@v7`, `actions/checkout@v5`, `actions/setup-node@v4`.

Locations:

- `.github/workflows/demo.yml:23`
- `.github/workflows/demo.yml:30`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:22`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:38`
- `.github/workflows/tests.yml:39`
- `.github/workflows/tests.yml:43`
- `.github/workflows/tests.yml:50`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:52`
- `.github/workflows/tests.yml:60`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and no job-level `permissions:` on any job, so the workflow runs with the default (potentially broad) token permissions. All jobs in this file are missing permissions blocks.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has a `permissions:` block only on the `commit-and-push-fresh-dist` job. The remaining jobs (`gitleaks`, `eslint`, `dist-built`, `run-this-action`) have no `permissions:` block and no top-level permissions key, so they run with default token permissions.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across three workflow files:

1. release.yml - script-injection: Moved `${{ github.actor }}` to an `env:` block as `GIT_ACTOR` and referenced it as `"$GIT_ACTOR"` in the shell script.
2. release.yml - missing-permissions: Added `permissions: {}` at top level and `permissions: contents: write` at job level.
3. release.yml - unpinned-uses: Pinned `actions/checkout@v5` and `gacts/github-slug@v1` to full SHAs.
4. demo.yml - unpinned-uses: Pinned `actions/checkout@v5` and `peaceiris/actions-gh-pages@v4` to full SHAs.
5. tests.yml - unpinned-uses: Pinned all 9 unpinned action references to full SHAs (actions/checkout@v4, gacts/gitleaks@v1, actions/checkout@v5 ×4, actions/setup-node@v4 ×2, actions/upload-artifact@v5, actions/download-artifact@v6, stefanzweifel/git-auto-commit-action@v7).
6. tests.yml - missing-permissions: Added `permissions: {}` to gitleaks, eslint, dist-built, and run-this-action jobs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/release.yml line 26: moved `${{ secrets.GITHUB_TOKEN }}` from the `run:` shell command string into the step's `env:` block as `GH_TOKEN: "${{ secrets.GITHUB_TOKEN }}"`, and updated the shell command to reference it as `$GH_TOKEN` instead of the direct `${{ ... }}` interpolation.

