<!-- markdownlint-disable -->

# Hardening Report: gacts--directory-listing/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--directory-listing/v1.0.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In .github/workflows/release.yml, the expressions ${{ github.actor }} and ${{ secrets.GITHUB_TOKEN }} are interpolated directly inside a run: shell command. An attacker who can control the actor name (e.g. via a fork or crafted event) could inject shell metacharacters. Offending lines:
  `git config --local user.name "${{ github.actor }}"`
  `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
Fix: move these values into env: variables and reference them as quoted shell variables ("$GIT_ACTOR").

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### unpinned-uses (severity: high)

Multiple uses: references across all workflow files use mutable tag or branch refs instead of pinned 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if any referenced action's tag is moved or compromised.

.github/workflows/demo.yml:
  - actions/checkout@v7 (line 24)
  - peaceiris/actions-gh-pages@v4 (line 33)

.github/workflows/release.yml:
  - actions/checkout@v7 (line 15)
  - gacts/github-slug@v1 (line 16)

.github/workflows/tests.yml:
  - actions/checkout@v7 (lines 24, 42, 57, 67)
  - gacts/gitleaks@v1 (line 26)
  - actions/setup-node@v7 (lines 32, 43)
  - actions/upload-artifact@v7 (line 47)
  - actions/download-artifact@v8 (line 59)
  - stefanzweifel/git-auto-commit-action@v7 (line 61)

Fix: pin every uses: reference to a full 40-character commit SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.

Locations:

- `.github/workflows/demo.yml:24`
- `.github/workflows/demo.yml:33`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:26`
- `.github/workflows/tests.yml:32`
- `.github/workflows/tests.yml:43`
- `.github/workflows/tests.yml:47`
- `.github/workflows/tests.yml:57`
- `.github/workflows/tests.yml:59`
- `.github/workflows/tests.yml:61`
- `.github/workflows/tests.yml:67`

### missing-permissions (severity: medium)

Two workflow files lack a top-level permissions: key and have jobs without job-level permissions blocks, meaning those jobs run with the default (potentially broad) token permissions.

.github/workflows/release.yml: No top-level permissions: key and the update-git-tag job has no permissions: block. This job performs git push operations and should declare minimal required permissions (e.g. contents: write).

.github/workflows/tests.yml: No top-level permissions: key. The jobs gitleaks, eslint, dist-built, and run-this-action all lack job-level permissions: blocks. Only commit-and-push-fresh-dist has explicit permissions. All jobs without explicit permissions inherit the default token scope.

Fix: add a top-level `permissions: {}` (deny-all default) and grant only the minimum required permissions per job.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:

1. script-injection (release.yml): Moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from the run: shell block into the step's env: block as GIT_ACTOR and GIT_TOKEN, referencing them as shell variables in the script.

2. unpinned-uses (all workflow files): Pinned all 8 unique action references to their full 40-character commit SHAs using lookup_action_sha, preserving the original tag in a comment (e.g., `# v7`).

3. missing-permissions (release.yml and tests.yml): Added top-level `permissions: {}` deny-all defaults to both files, and added job-level permissions blocks with minimal required permissions (contents: read for read-only jobs, contents: write for jobs that push, pull-requests: write where needed).

