<!-- markdownlint-disable -->

# Hardening Report: gacts--directory-listing/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--directory-listing/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references across the workflow files are pinned to mutable tags or version strings rather than immutable 40-character SHA commit hashes. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references include: actions/checkout@v4, peaceiris/actions-gh-pages@v4, gacts/github-slug@v1, actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, actions/download-artifact@v4, stefanzweifel/git-auto-commit-action@v5, gacts/gitleaks@v1.

Locations:

- `.github/workflows/demo.yml:20`
- `.github/workflows/demo.yml:27`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:15`
- `.github/workflows/tests.yml:22`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:37`
- `.github/workflows/tests.yml:38`
- `.github/workflows/tests.yml:41`
- `.github/workflows/tests.yml:50`
- `.github/workflows/tests.yml:51`
- `.github/workflows/tests.yml:53`
- `.github/workflows/tests.yml:61`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level permissions: block, and most jobs also lack job-level permissions. demo.yml has no permissions at any level. release.yml has no permissions at any level. tests.yml has permissions only on the commit-and-push-fresh-dist job; the gitleaks, eslint, dist-built, and run-this-action jobs all lack permissions blocks. Without explicit permissions, workflows run with the default (potentially write-all) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/demo.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

### script-injection (severity: high)

Sub-rule (a): In release.yml, the expression ${{ github.actor }} is interpolated directly inside a run: shell command string. This means the value is substituted into the shell script before the shell parses it, allowing an attacker who controls their GitHub username to inject arbitrary shell commands. Offending lines: `git config --local user.name "${{ github.actor }}"` and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`. These should be moved to an env: block and the env var should be double-quoted in the shell script.

Locations:

- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across demo.yml, release.yml, and tests.yml:

1. unpinned-uses: Pinned all 8 unique action references to full 40-char SHA hashes (actions/checkout, peaceiris/actions-gh-pages, gacts/github-slug, actions/setup-node, actions/upload-artifact, actions/download-artifact, stefanzweifel/git-auto-commit-action, gacts/gitleaks), preserving the original tag as a comment.

2. missing-permissions: Added top-level `permissions: {}` to all three workflow files. Added job-level permissions to all jobs: read-only jobs get `contents: read`; jobs that push to git get `contents: write`; the demo job also gets `pages: write`; commit-and-push-fresh-dist retains its existing `contents: write` and `pull-requests: write`.

3. script-injection: In release.yml, moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` from the inline `run:` shell string into the step's `env:` block as `GIT_ACTOR` and `GITHUB_TOKEN` respectively. The shell script now references them as plain env vars, eliminating the injection vector.

