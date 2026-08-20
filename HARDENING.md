<!-- markdownlint-disable -->

# Hardening Report: tj-actions--branch-names/v9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--branch-names/v9** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in test.yml directly interpolate `${{ steps.*.outputs.* }}` expressions inside shell commands. For example: `echo "Default Branch: ${{ steps.branch-names.outputs.default_branch }}"` and `head_ref="${{ steps.branch-names-replace-slashes.outputs.head_ref_branch }}"`. Any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because YAML template substitution happens before the shell ever sees the value, allowing shell metacharacters to be injected. These step outputs are derived from attacker-controllable branch/ref names.

Locations:

- `.github/workflows/test.yml:31`
- `.github/workflows/test.yml:107`

### github-env-injection (severity: high)

The `current_branch` step in action.yml writes untrusted values to $GITHUB_OUTPUT without sanitization: `echo "current_branch=$HEAD_REF_BRANCH" >> "$GITHUB_OUTPUT"` and `echo "current_branch=$REF_BRANCH" >> "$GITHUB_OUTPUT"`. The env vars HEAD_REF_BRANCH and REF_BRANCH are sourced from `steps.branch.outputs.head_ref_branch` and `steps.branch.outputs.ref_branch`, which are themselves derived from `github.head_ref` and `github.ref` (attacker-controllable). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection into the output file.

Locations:

- `action.yml:94`

### github-env-injection (severity: high)

The `default` step in action.yml writes `echo "default_branch=$DEFAULT_BRANCH" >> "$GITHUB_OUTPUT"` (twice, in both branches of the if/else) without sanitization. The env var DEFAULT_BRANCH is sourced from `github.event.repository.default_branch` (workflow-controlled). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection into the output file.

Locations:

- `action.yml:110`
- `action.yml:112`

### missing-permissions (severity: medium)

The workflow file test.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

The workflow file codacy-analysis.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Explicit minimal permissions (e.g., `security-events: write` for SARIF upload) should be declared.

Locations:

- `.github/workflows/codacy-analysis.yml:1`

### unpinned-uses (severity: high)

The following `uses:` references in codacy-analysis.yml are pinned to mutable tag refs rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved: `codacy/codacy-analysis-cli-action@v4.4.7` (line 35) and `github/codeql-action/upload-sarif@v3` (line 50). Each should be pinned to a full SHA digest.

Locations:

- `.github/workflows/codacy-analysis.yml:35`
- `.github/workflows/codacy-analysis.yml:50`

### unpinned-uses (severity: high)

The following `uses:` references in update-readme.yml are pinned to mutable tag refs rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks: `tj-actions/auto-doc@v3` (line 21), `tj-actions/remark@v3` (line 27), `tj-actions/verify-changed-files@v20` (line 30), and `peter-evans/create-pull-request@v7` (line 43). Each should be pinned to a full SHA digest.

Locations:

- `.github/workflows/update-readme.yml:21`
- `.github/workflows/update-readme.yml:27`
- `.github/workflows/update-readme.yml:30`
- `.github/workflows/update-readme.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all 7 findings across 4 files:

1. **script-injection** (test.yml lines 31, 107 and all other occurrences): Moved all `${{ steps.*.outputs.* }}` expressions from `run:` blocks into `env:` blocks, referencing them as plain env vars in shell scripts.

2. **github-env-injection** (action.yml lines 94, 110, 112): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing HEAD_REF_BRANCH, REF_BRANCH, and DEFAULT_BRANCH to $GITHUB_OUTPUT in the `current_branch` and `default` steps.

3. **missing-permissions** (test.yml): Added top-level `permissions: contents: read`.

4. **missing-permissions** (codacy-analysis.yml): Added top-level `permissions: contents: read` and `security-events: write`.

5. **unpinned-uses** (codacy-analysis.yml): Pinned `codacy/codacy-analysis-cli-action@562ee3e...` and `github/codeql-action/upload-sarif@f371297...`.

6. **unpinned-uses** (update-readme.yml): Pinned `tj-actions/auto-doc@b10ceed...`, `tj-actions/remark@10fc407...`, `tj-actions/verify-changed-files@a1c6ace...`, and `peter-evans/create-pull-request@22a9089...`.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:

1. **github-env-injection (branch step)**: Added `safe_base=$(printf '%s' "$BASE_REF" | tr -d '\n\r')`, `safe_head=$(printf '%s' "$HEAD_REF" | tr -d '\n\r')`, and `safe_ref=$(printf '%s' "$REF_BRANCH" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT for the non-tag branch. Similarly added `safe_base` sanitization for the tag branch of the `branch` step.

2. **github-env-injection (tag step)**: Added `safe_tag=$(printf '%s' "$TAG" | tr -d '\n\r')` before writing `tag=` to $GITHUB_OUTPUT.

3. **script-injection**: Quoted the prefix variables inside bash pattern substitutions: changed `${REF_BRANCH/$INPUTS_STRIP_BRANCH_PREFIX/}` to `${REF_BRANCH/"$INPUTS_STRIP_BRANCH_PREFIX"/}`, `${HEAD_REF/$INPUTS_STRIP_BRANCH_PREFIX/}` to `${HEAD_REF/"$INPUTS_STRIP_BRANCH_PREFIX"/}`, `${BASE_REF/refs\/heads\/$INPUTS_STRIP_TAG_PREFIX/}` to `${BASE_REF/refs\/heads\/"$INPUTS_STRIP_TAG_PREFIX"/}`, and `${REF/refs\/tags\/$INPUTS_STRIP_TAG_PREFIX/}` to `${REF/refs\/tags\/"$INPUTS_STRIP_TAG_PREFIX"/}`. Quoting the pattern in bash parameter expansion prevents glob metacharacter interpretation.

