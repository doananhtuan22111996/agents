# Mobile: Create snapshot branch for CI publishing

Create a snapshot branch from the current feature branch so the CI pipeline auto-publishes a snapshot version.

## Safety

- Only push to `snapshot/*` branches — never to `master`, `main`, or `release/*`
- Only push to the current repo's remote — verify remote URL matches expected repo
- Never force push

## Steps

### 1. Verify current context

```bash
BRANCH_NAME=$(git rev-parse --abbrev-ref HEAD)
REMOTE_URL=$(git remote get-url origin)
```

- Verify this is a working branch (e.g., `feature/PNL-XXXXX-...`, `bugfix/PNL-XXXXX-...`). If on `master`, `main`, or `release/*`, warn the user and abort.
- Print the remote URL so user can verify correct repo.

### 2. Create and push snapshot branch

**IMPORTANT**: Strip any branch prefix (`feature/`, `bugfix/`, `hotfix/`, `fix/`, etc.) from the branch name. The snapshot branch should be `snapshot/PNL-xxx-description`, NOT `snapshot/feature/PNL-xxx-description`.

```bash
# Strip known branch prefixes for snapshot naming
SNAPSHOT_NAME=$(echo "$BRANCH_NAME" | sed 's|^feature/||;s|^bugfix/||;s|^hotfix/||;s|^fix/||')
git checkout -b "snapshot/$SNAPSHOT_NAME"
git push origin "snapshot/$SNAPSHOT_NAME"
```

### 3. Switch back to original branch

```bash
git checkout $BRANCH_NAME
```

### 4. Log output

```
=== [mobile-snapshot] ===
Status: PASSED | FAILED
Repo: <remote URL>
Feature branch: <branch name>
Snapshot branch: snapshot/<branch name>
Next: check pipeline for published snapshot version
```
