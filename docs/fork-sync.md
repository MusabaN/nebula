# Keeping this fork in sync with upstream

This repository is a fork of [AgentSystemLabs/nebula](https://github.com/AgentSystemLabs/nebula).
The **Sync upstream** workflow (`.github/workflows/sync-upstream.yml`) keeps it current.

## What happens

Every day at 05:17 UTC, and whenever you run it by hand from the Actions tab:

1. The workflow fetches upstream's `main`. If this fork already has every commit, it stops.
2. Otherwise it pushes those commits to the `upstream-sync` branch here.
3. It opens a pull request from `upstream-sync` into `main`, unless one is already open.

Nothing reaches `main` without you. The pull request runs CI and shows whether upstream's changes
conflict with the fork's own.

## Merging the pull request

**Use "Create a merge commit".** Do not squash or rebase. The merge commit is how the next sync
knows which upstream commits are already in `main`; after a squash, every later sync shows the
same commits again and conflicts with them.

If GitHub reports conflicts, resolve them on the `upstream-sync` branch, either with the
"Resolve conflicts" button or locally:

```bash
git fetch origin
```

```bash
git switch upstream-sync
```

```bash
git merge origin/main
```

Fix the files, commit, and push. Later runs keep your resolution: they merge new upstream commits
into the branch rather than overwrite it. If a new upstream commit conflicts with what is already
on the branch, the run fails and says so, and you merge upstream in by hand:

```bash
git remote add upstream https://github.com/AgentSystemLabs/nebula.git
```

```bash
git fetch upstream main
```

```bash
git merge upstream/main
```

## One-time setup

- **Enable Actions on the fork.** GitHub turns workflows off in a new fork. Open the Actions tab
  and enable them, then run **Sync upstream** once by hand to confirm it works.
- **Allow Actions to open pull requests.** Settings → Actions → General → Workflow permissions →
  "Allow GitHub Actions to create and approve pull requests".
- **Recommended: add an `UPSTREAM_SYNC_TOKEN` secret.** The built-in token has two limits:
  it cannot push commits that change files under `.github/workflows`, and pull requests it
  opens do not start CI. A fine-grained personal access token for this repository with
  **Contents**, **Pull requests** and **Workflows** set to read and write lifts both. Without it
  the sync still works until upstream edits a workflow file; that run then fails with a message
  pointing here, and for CI you close and reopen the pull request.

GitHub pauses scheduled workflows in a repository with no activity for 60 days. Merging the sync
pull requests counts as activity.

## Changing the schedule

Edit the `cron` line in the workflow. For a weekly sync on Mondays:

```yaml
- cron: "17 5 * * 1"
```

## What the fork changes

The smaller the difference from upstream, the fewer conflicts. Today the fork adds:

- GitHub Copilot CLI as a built-in harness. See
  [GitHub Copilot CLI](configuration.md#github-copilot-cli).
- This sync workflow and document.

The Copilot change touches lines that list every harness (the `AgentKind` enum, the registry
tests, the harness lists in the docs). Upstream edits those same lines when it adds a harness of
its own, so that is where a conflict is most likely; keep both entries when it happens.
