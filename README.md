# GEOFlow fork sync

This repository keeps `zacharydigital/GEOFlow`'s `main` branch aligned with
`yaojingang/GEOFlow`'s `main` branch.

## Safety model

- The target branch is updated with `git merge --ff-only`.
- The workflow never force-pushes or overwrites divergent commits.
- If custom commits are added to `zacharydigital/GEOFlow:main`, synchronization fails visibly.
- Product development belongs on `develop` and `feature/*` branches, not `main`.
- GEOFlow's in-application updater is unrelated to this Git synchronization.

## One-time setup

The token used by this repository's built-in `GITHUB_TOKEN` cannot write to a
different repository. Create a fine-grained personal access token for the target
repository:

1. Open GitHub **Settings → Developer settings → Personal access tokens → Fine-grained tokens**.
2. Limit repository access to **Only select repositories → `zacharydigital/GEOFlow`**.
3. Grant repository permission **Contents: Read and write**. Metadata read access is automatic.
4. In this repository, open **Settings → Secrets and variables → Actions**.
5. Create a repository secret named exactly `FORK_SYNC_TOKEN`.

Never put the token in a workflow file, commit, issue, or log.

## Run and schedule

The workflow is at [`.github/workflows/sync-geoflow.yml`](.github/workflows/sync-geoflow.yml).

It runs:

- manually through **Actions → Sync GEOFlow fork → Run workflow**;
- every six hours at 23 minutes past the hour (UTC);
- a small monthly keepalive commit because this automation repository is public
  and GitHub may disable scheduled workflows after 60 days without repository
  activity.

The synchronization job:

1. authenticates with `FORK_SYNC_TOKEN`;
2. clones `zacharydigital/GEOFlow:main`;
3. fetches `yaojingang/GEOFlow:main` as `upstream`;
4. fast-forwards only;
5. pushes `main` to the target repository;
6. verifies that the target and upstream commit SHAs are identical.

## Failure handling

A failed fast-forward indicates that the target `main` contains commits not
present in upstream. Do not add `--force` to the workflow. Move custom work to a
development branch, restore `main` intentionally, and rerun the workflow.

Token expiration or revocation also causes a visible workflow failure. Rotate
the token and replace `FORK_SYNC_TOKEN` without changing the workflow.
