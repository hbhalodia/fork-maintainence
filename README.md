# Fork Sync Automation

This repository contains a GitHub Actions workflow that automatically syncs selected forked repositories with their upstream repositories.

It is useful for open source contributors who maintain many forked repositories and want their forks to stay updated without manually clicking **Sync fork** on GitHub.

## What This Does

The workflow:

- Runs on a configured schedule.
- Finds forked repositories accessible by the configured GitHub token.
- Syncs each fork's default branch with its upstream repository.
- Prints readable logs for each repository.
- Reports successful, failed, and skipped sync attempts.
- Can be triggered manually from the GitHub Actions UI.

After the workflow runs successfully, local repositories can be updated with:

```bash
git pull origin main
```

or with the repository's default branch:

```bash
git pull origin trunk
```

```bash
git pull origin master
```

## Repository Structure

Create the workflow file at:

```text
.github/workflows/sync-forks.yml
```

The full workflow YAML should live in that file, not in this README.

## Token Setup

Create a fine-grained personal access token from:

```text
GitHub -> Settings -> Developer settings -> Personal access tokens -> Fine-grained tokens
```

Recommended token name:

```text
FORK_SYNC_TOKEN
```

Recommended repository access:

```text
Selected repositories
```

Select the forked repositories that should be synced.

Required permissions:

```text
Repository permissions:
  Contents: Read and write
  Workflows: Read and write
  Metadata: Read-only
```

The `Workflows: Read and write` permission is needed because upstream repositories may update files inside:

```text
.github/workflows/
```

Without this permission, GitHub may fail with an error like:

```text
refusing to allow a Personal Access Token to create or update workflow without workflow scope
```

## Add Token To Repository Secrets

In this maintenance repository, go to:

```text
Settings -> Secrets and variables -> Actions -> New repository secret
```

Add the secret:

```text
Name: FORK_SYNC_TOKEN
Value: your fine-grained token
```

## Schedule

GitHub Actions cron schedules use UTC.

### Run Every 5 Minutes For Testing

```yaml
- cron: "*/5 * * * *"
```

### Run Daily At 9:00 AM IST

```yaml
- cron: "30 3 * * *"
```

Explanation:

```text
09:00 IST = 03:30 UTC
```

Recommended production schedule:

```yaml
on:
  schedule:
    - cron: "30 3 * * *"
  workflow_dispatch:
```

Keep `workflow_dispatch` enabled so the workflow can be triggered manually.

## How Repository Selection Works

The workflow uses the GitHub API to fetch repositories visible to the configured token.

Because the token is scoped to selected repositories, the workflow only sees and syncs those selected repositories.

The workflow also filters repositories so only forked repositories are synced. Non-fork repositories are ignored.

## Logging

The workflow logs include:

- Start time.
- Repository name.
- Default branch.
- Sync status.
- Success or failure result.
- Failure reason, when detectable.
- Final summary with total, synced, failed, and skipped repositories.

Example log format:

```text
============================================================
Fork Sync Started
Started at: 2026-05-22 03:30:00 UTC
============================================================

------------------------------------------------------------
Repository : hbhalodia/example-repo
Branch     : main
Status     : Syncing
Started    : 2026-05-22 03:30:02 UTC
Result     : SUCCESS
Message    : Fork synced successfully
Finished   : 2026-05-22 03:30:04 UTC
------------------------------------------------------------

============================================================
Fork Sync Summary
Finished at : 2026-05-22 03:30:10 UTC
Total forks : 12
Synced      : 11
Failed      : 1
Skipped     : 0
============================================================
```

## Known Failure Cases

### Repository Rule Violations

Example error:

```text
Repository rule violations found
```

This means the fork has a branch rule, branch protection rule, or ruleset that prevents the sync.

Check:

```text
Repository -> Settings -> Rules -> Rulesets
```

and:

```text
Repository -> Settings -> Branches -> Branch protection rules
```

For personal forks, the default branch usually should allow direct updates from your automation token.

### Missing Workflow Permission

Example error:

```text
refusing to allow a Personal Access Token to create or update workflow without workflow scope
```

Fix this by adding the following permission to the fine-grained token:

```text
Workflows: Read and write
```

### Merge Conflicts

If the fork has custom commits that conflict with upstream, GitHub cannot automatically sync it.

In that case, sync the repository manually and resolve conflicts.

## Manual Run

To run the workflow manually:

```text
Actions -> Sync forked repositories -> Run workflow
```

This is useful for testing after changing token permissions or updating the workflow.

## Local Usage

Once the scheduled sync has run successfully, update your local repository with:

```bash
git pull origin main
```

or:

```bash
git pull origin trunk
```

or:

```bash
git pull origin master
```

depending on the repository's default branch.

## Notes

- This workflow syncs each fork's default branch.
- GitHub Actions schedules may start a few minutes late during busy periods.
- If a repository fails to sync, the workflow logs should show the reason and suggested fix.
- The workflow exits with failure if one or more repositories could not be synced.
