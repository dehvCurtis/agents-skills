---
name: soliditybom-ship
description: Full shipping pipeline — version bump, update docs, security checks, tests, commit, PR, merge, tag, release, and verify for SolidityBOM.
user_invocable: true
---

# /ship — SolidityBOM Shipping Pipeline

Run this skill when the user types `/ship`. It executes the full shipping process in order. Each phase must pass before proceeding to the next. Ask the user for confirmation before creating PRs and before merging.

The user may pass an optional argument describing what's being shipped (e.g., `/ship fix project detector false-positive`). Use this to inform commit messages, PR descriptions, and doc updates.

## Ownership Rules (ALWAYS ENFORCED)

- Git user is `dehvCurtis` — all commits, PRs, and pushes use this account.
- **NEVER** add `Co-Authored-By` trailers or any Claude/AI attribution to commits, PRs, branches, or comments.
- **NEVER** mention Claude Code, AI, or automated tooling in PR descriptions or commit messages.
- Each destructive git action (force push, reset) requires explicit user approval.

## Repo

| Repo | Path | Remote |
|------|------|--------|
| SolidityBOM | `/home/pwner/Git/SolidityBOM` | `BlockSecOps/SolidityBOM` |

---

## Phase 1: Version Bump and Documentation

### 1a. Bump version in `solidity-sbom/Cargo.toml`

- Update the `version` field in `solidity-sbom/Cargo.toml`.
- Follow semver: patch for fixes, minor for new features, major for breaking changes.
- Ask the user to confirm the new version number before proceeding.

### 1b. Update `README.md` version badge and any version references

- Update the `[![Version](...)` badge in `README.md` to the new version.
- Search for any other hardcoded version strings and update them.

### 1c. Update `CHANGELOG.md` or release notes if one exists

- Add a new version entry at the top describing what changed.
- Summarize every change being shipped with enough detail for users to understand impact.

### 1d. Update any docs in `docs/` that are affected

- If CLI behavior changed, update relevant docs.
- If new flags or options were added, document them.

---

## Phase 2: Security Checks

Run these checks and report results. All must pass.

### 2a. Build release binary

```bash
cd /home/pwner/Git/SolidityBOM/solidity-sbom && cargo build --release 2>&1
```

If build fails, stop and fix before continuing.

### 2b. Run full test suite

```bash
cd /home/pwner/Git/SolidityBOM/solidity-sbom && cargo test --workspace --lib 2>&1
```

Report total test count and pass/fail. Must be 0 failures.

### 2c. Check for credential/secret leaks

```bash
cd /home/pwner/Git/SolidityBOM && git diff --name-only HEAD | xargs grep -l -i -E '(password|secret|api_key|private_key|token\s*=)' 2>/dev/null || echo "No secrets found"
```

### 2d. Check for unsafe code

```bash
grep -rn "unsafe " /home/pwner/Git/SolidityBOM/solidity-sbom/src --include="*.rs" | grep -v "// SAFETY:" | head -20
```

Any new `unsafe` blocks without `// SAFETY:` comments must be reviewed.

---

## Phase 3: Functional Verification

### 3a. Run integration tests (if present)

```bash
cd /home/pwner/Git/SolidityBOM/solidity-sbom && cargo test --test '*' 2>&1 | tail -10
```

### 3b. Smoke test the binary on the example contracts

```bash
/home/pwner/Git/SolidityBOM/solidity-sbom/target/release/solidity-sbom analyze /home/pwner/Git/SolidityBOM/examples --multi-version 2>&1
```

Must complete without error and report a non-zero component count.

### 3c. Verify project detection still works

```bash
/home/pwner/Git/SolidityBOM/solidity-sbom/target/release/solidity-sbom --version 2>&1
```

---

## Phase 4: Commit, PR, and Merge

### 4a. Check for changes

```bash
cd /home/pwner/Git/SolidityBOM && git status && git diff --stat
```

### 4b. Create branch (if not already on a feature branch)

```bash
git checkout -b <branch-name>
```

Branch naming: `fix/<description>`, `feat/<description>`, `docs/<description>`, or `release/v<version>`.

### 4c. Stage and commit

- Stage only relevant files (never `git add -A` without reviewing first).
- Write a clear commit message summarizing the changes. NO Claude attribution.
- Example format:
  ```
  fix: remove lib/ directory as Foundry project indicator

  - is_foundry_project() was matching /lib on Linux, misrouting any
    project without foundry.toml as a Foundry project rooted at /
  - Standalone detection now runs before parent-directory walk
  - has_solidity_files() now recurses up to 4 directory levels
  ```

### 4d. Push branch

```bash
git push -u origin <branch-name>
```

### 4e. Create PR

```bash
gh pr create --title "<title>" --body "<description>"
```

PR description format:
```
## Summary
- Bullet point summary of changes

## Changes
- Detailed list of what changed and why

## Testing
- [ ] All unit tests pass (N/N)
- [ ] Binary smoke test passes on example contracts
- [ ] No secrets in diff
- [ ] No new unsafe blocks

## Notes
Any caveats, known issues, or follow-up items.
```

**Do NOT mention Claude Code, AI, or automated tools anywhere in the PR.**

### 4f. Ask user to confirm merge, then

```bash
gh pr merge <pr-number> --merge
```

### 4g. Return to main

```bash
git checkout main && git pull origin main
```

---

## Phase 5: Tag and Release

Only proceed after Phase 4 merge is complete and main is up to date.

### 5a. Create and push git tag

```bash
cd /home/pwner/Git/SolidityBOM && git checkout main && git pull origin main
git tag v<version>
git push origin v<version>
```

Tag must match the version in `solidity-sbom/Cargo.toml`. Ask the user to confirm before pushing the tag.

### 5b. Verify CI release

Wait for the GitHub Actions release workflow to complete, then verify:

```bash
gh run list --workflow=release.yml --limit 1
gh release view v<version>
```

Confirm:
- CI workflow succeeded
- Release is published (not draft)
- Platform binaries are attached
- Release notes reflect the changelog entry

Report the release URL to the user. If CI failed, investigate with `gh run view <run-id> --log-failed`.

---

## Error Handling

- If any phase fails, stop and report the failure clearly.
- Do not skip phases or continue past failures.
- If a PR fails CI, investigate and fix before re-attempting.
- If merge conflicts occur, resolve them and ask for user confirmation before force-resolving.
