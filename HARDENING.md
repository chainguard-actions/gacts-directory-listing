<!-- markdownlint-disable -->

# Hardening Report: gacts--directory-listing/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--directory-listing/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use mutable tag-based `uses:` references instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks. Failing references: demo.yml — `actions/checkout@v4`, `peaceiris/actions-gh-pages@v4`; release.yml — `actions/checkout@v4`, `gacts/github-slug@v1`; tests.yml — `actions/checkout@v4` (multiple), `gacts/gitleaks@v1`, `actions/setup-node@v4` (multiple), `actions/upload-artifact@v4`, `actions/download-artifact@v4`, `stefanzweifel/git-auto-commit-action@v5`.

Locations:

- `.github/workflows/demo.yml:21`
- `.github/workflows/demo.yml:28`
- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:13`
- `.github/workflows/tests.yml:22`
- `.github/workflows/tests.yml:27`
- `.github/workflows/tests.yml:33`
- `.github/workflows/tests.yml:36`
- `.github/workflows/tests.yml:44`
- `.github/workflows/tests.yml:50`
- `.github/workflows/tests.yml:53`
- `.github/workflows/tests.yml:60`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and most jobs also lack job-level `permissions:` blocks, leaving them with default (potentially broad) GITHUB_TOKEN permissions. demo.yml: no top-level or job-level permissions on the `demo` job. release.yml: no top-level or job-level permissions on the `update-git-tag` job. tests.yml: no top-level permissions; only `commit-and-push-fresh-dist` has job-level permissions — `gitleaks`, `eslint`, `dist-built`, and `run-this-action` jobs do not.

Locations:

- `.github/workflows/demo.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

### script-injection (severity: high)

Sub-rule (a): release.yml directly interpolates `${{ github.actor }}` inside a `run:` shell command. The Actions runner substitutes the expression before the shell parses it, so a crafted actor name containing shell metacharacters could inject arbitrary commands. Offending lines: `git config --local user.name "${{ github.actor }}"` and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`.

Locations:

- `.github/workflows/release.yml:16`
- `.github/workflows/release.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across demo.yml, release.yml, and tests.yml:

1. unpinned-uses: Pinned all 8 distinct action references to their full 40-character SHA commit hashes (with original tag preserved as inline comment): actions/checkout@11d5960a, peaceiris/actions-gh-pages@84c30a85, gacts/github-slug@83cd3d95, gacts/gitleaks@c9a0338, actions/setup-node@49933ea5, actions/upload-artifact@ea165f8d, actions/download-artifact@d3f86a10, stefanzweifel/git-auto-commit-action@b863ae19.

2. missing-permissions: Added top-level `permissions: {}` to all three workflow files. Added job-level permissions to all jobs that lacked them: gitleaks/eslint/dist-built/run-this-action got `contents: read`; update-git-tag got `contents: write`; demo got `contents: write` and `pages: write`.

3. script-injection: In release.yml, moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from inline run: shell strings into the step's env: block as GIT_ACTOR and GIT_TOKEN respectively. The shell script now references them as plain environment variables, eliminating the injection risk.

