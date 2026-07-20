[![StepSecurity Maintained Action](https://raw.githubusercontent.com/step-security/maintained-actions-assets/main/assets/maintained-action-banner.png)](https://docs.stepsecurity.io/actions/stepsecurity-maintained-actions)

# cocogitto-action

A GitHub Action that installs [cocogitto](https://github.com/cocogitto/cocogitto) and optionally enforces [Conventional Commits](https://conventionalcommits.org/) or automates versioned releases.

## Prerequisites

`actions/checkout` must run with `fetch-depth: 0` before this action. Without the full history cocogitto cannot walk commits or determine the next version.

## Usage

### Enforce commit conventions

```yaml
on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - uses: step-security/cocogitto-action@v4
        with:
          command: check
```

## Inputs

| Input            | Description                                                        | Required | Default                                          |
|------------------|--------------------------------------------------------------------|----------|--------------------------------------------------|
| `command`        | Cocogitto subcommand to run (e.g. `check`, `bump`, `changelog`)   | No*      | —                                                |
| `args`           | Extra flags passed to the subcommand                               | No       | `""`                                             |
| `install-only`   | Install cocogitto and add to PATH without running a command        | No       | `false`                                          |
| `git-user`       | Value for `git config user.name`                                   | No       | `github-actions`                                 |
| `git-user-email` | Value for `git config user.email`                                  | No       | `github-actions[bot]@users.noreply.github.com`   |

\* Required when `install-only` is not `true`.

## Outputs

| Output    | Description                                      |
|-----------|--------------------------------------------------|
| `version` | New version tag after a `bump` or `release` run  |
| `stdout`  | Full standard output from the cocogitto command  |

## More examples

### Pull request checks

Check out the PR head instead of the merge commit:

```yaml
on: [pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0
          ref: ${{ github.event.pull_request.head.sha }}

      - uses: step-security/cocogitto-action@v4
        with:
          command: check
```

### Check only since the latest tag

If older commits in your history are not conventional-commit compliant, scope the check to everything after the most recent tag:

```yaml
      - uses: step-security/cocogitto-action@v4
        with:
          command: check
          args: --from-latest-tag
```

Given this history:

```
* 9b609bc - (HEAD -> main) WIP: feat unfinished work
* d832ca4 - feat: working on feature A
* d5ce110 - (tag: 0.1.0) chore: release 0.1.0
* 8f25a4b - chore: a commit before tag 0.1.0
```

Only the two commits after `0.1.0` are checked. The action fails on the `WIP:` commit at HEAD.

### Automated release

```yaml
      - uses: step-security/cocogitto-action@v4
        id: release
        with:
          command: bump
          args: --auto
          git-user: 'Cog Bot'
          git-user-email: 'bot@example.com'

      - name: Print released version
        run: echo "${{ steps.release.outputs.version }}"
```

### Generate a changelog

```yaml
name: Publish Release

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0

      - uses: step-security/cocogitto-action@v4
        id: changelog
        with:
          command: changelog
          args: --at ${{ github.ref_name }}

      - uses: step-security/action-gh-release@v3
        with:
          tag_name: ${{ github.ref_name }}
          body: ${{ steps.changelog.outputs.stdout }}
```

### Install only

Set `install-only: true` to put the `cog` binary on `PATH` without running any command. Subsequent steps can then call `cog` directly.

```yaml
      - uses: step-security/cocogitto-action@v4
        with:
          install-only: true

      - run: cog check
```

## Notes

- After this action completes, `cog` remains available on `PATH` for the rest of the job.
- Set `git-user` and `git-user-email` when running `bump` so release commits are attributed correctly.
