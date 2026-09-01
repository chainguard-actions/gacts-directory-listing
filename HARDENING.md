<!-- markdownlint-disable -->

# Hardening Report: gacts--directory-listing/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--directory-listing/v1.0.5** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: Direct expression interpolation inside a run: block. In release.yml, the step that configures git uses `${{ github.actor }}` directly inside the shell command string in two places:
  - `git config --local user.name "${{ github.actor }}"`
  - `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
An attacker who can control the actor name (e.g. via a crafted username) could inject shell metacharacters. These values should be moved to env: variables and referenced as `"$VAR"` in the run: block.

Locations:

- `.github/workflows/release.yml:18`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and its only job (`update-git-tag`) also has no job-level `permissions:` block. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. A minimal permissions block (e.g. `contents: write`) should be added.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level `permissions:` key, and the jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` have no job-level `permissions:` blocks. Only `commit-and-push-fresh-dist` defines job-level permissions. A top-level `permissions: {}` (read-only) block should be added, with specific overrides per job as needed.

Locations:

- `.github/workflows/tests.yml:1`

### unpinned-uses (severity: high)

All `uses:` references in the workflow files use mutable tag or version refs instead of full 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if those tags are moved or compromised.

demo.yml:
  - `actions/checkout@v7`
  - `peaceiris/actions-gh-pages@v4`

release.yml:
  - `actions/checkout@v7`
  - `gacts/github-slug@v1`

tests.yml:
  - `actions/checkout@v7` (multiple)
  - `actions/setup-node@v7` (multiple)
  - `gacts/gitleaks@v1`
  - `actions/upload-artifact@v7`
  - `actions/download-artifact@v8`
  - `stefanzweifel/git-auto-commit-action@v7`

All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/demo.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all findings across three workflow files:

1. release.yml - script-injection: Moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from inline run: shell strings into the step's env: block as GIT_ACTOR and GITHUB_TOKEN, then referenced as plain shell variables.

2. release.yml - missing-permissions: Added top-level `permissions: contents: write` (needed for pushing tags).

3. tests.yml - missing-permissions: Added top-level `permissions: {}` to restrict default token permissions; the commit-and-push-fresh-dist job already had its own job-level permissions override.

4. All three workflow files - unpinned-uses: Pinned all 8 action references to full 40-character commit SHAs with the original tag preserved in a comment:
   - actions/checkout@v7 → 3d3c42e5aac5ba805825da76410c181273ba90b1
   - peaceiris/actions-gh-pages@v4 → 84c30a85c19949d7eee79c4ff27748b70285e453
   - gacts/github-slug@v1 → c29c0ddd888a1703a6fc06c55a5f6ddd7beae490
   - actions/setup-node@v7 → 820762786026740c76f36085b0efc47a31fe5020
   - gacts/gitleaks@v1 → 4fd785dcdaf557fda64e05b0ae033b8f7d82fb81
   - actions/upload-artifact@v7 → 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - actions/download-artifact@v8 → 3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c
   - stefanzweifel/git-auto-commit-action@v7 → 4a55954c782fc1ea30b9056cd3e7a2b40ca8887d

