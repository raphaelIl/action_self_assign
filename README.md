# Self Assign PR Author

A reusable GitHub Action that automatically assigns the pull request author as an assignee when a PR is opened.

## Usage

Add the following workflow to your repository at `.github/workflows/self-assign.yml`. Replace `<OWNER>` with your own GitHub account or organization that hosts this action.

```yaml
name: Self Assign

on:
  pull_request:
    types: [opened]

permissions:
  pull-requests: write

jobs:
  self-assign:
    runs-on: ubuntu-latest
    steps:
      - uses: <OWNER>/action_self_assign@v1
        with:
          # Optional. Defaults to the automatic GITHUB_TOKEN.
          repo-token: ${{ github.token }}
          # Optional. Set to "true" to skip PRs opened by bots.
          skip-bots: 'false'
```

## Inputs

| Name         | Description                                                                                          | Required | Default               |
| ------------ | ---------------------------------------------------------------------------------------------------- | -------- | --------------------- |
| `repo-token` | GitHub token used to assign the PR author. Defaults to the workflow's automatic `GITHUB_TOKEN`.      | No       | `${{ github.token }}` |
| `skip-bots`  | When `"true"`, pull requests opened by bot accounts (e.g. `dependabot[bot]`) are skipped.            | No       | `'false'`             |

## Permissions

This action calls the GitHub REST API to add an assignee, which requires write access to pull requests. Grant it in your workflow:

```yaml
permissions:
  pull-requests: write
```

If your workflow already sets a broader `permissions` block, make sure `pull-requests: write` is included.

## Fork pull requests

The token's effective permissions depend on which event triggers the workflow.

When the workflow runs on the `pull_request` event, the `GITHUB_TOKEN` for a PR opened **from a fork** is read-only. In that case the assignment call fails and the action reports an error. PRs created from a branch **within the same repository** work normally because the token retains write access.

To also handle PRs opened from forks, trigger the workflow with the `pull_request_target` event instead. With `pull_request_target` the token runs with write permissions even for fork PRs. This action never checks out the PR's code, so it does not execute untrusted contributor code — which makes `pull_request_target` comparatively safe here. Still, review the [GitHub security guidance](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/) before adopting it broadly.

| Event                  | Same-repo PRs | Fork PRs                  | Notes                                                                 |
| ---------------------- | ------------- | ------------------------- | --------------------------------------------------------------------- |
| `pull_request`         | Works         | Fails (token is read-only)| Safest default; sufficient when contributors push branches directly. |
| `pull_request_target`  | Works         | Works (token has write)   | Needed for fork PRs. Safe here because PR code is never checked out.  |

## Behavior

- **Idempotent**: if the author is already an assignee, the action does nothing and makes no API call.
- **Additive**: existing assignees are kept; the author is only added, never replacing others.
- **Bot filtering**: set `skip-bots: 'true'` to skip pull requests opened by bot accounts.
- **Event guard**: if invoked without a `pull_request` payload, the action fails with a clear message.

## License

[MIT](./LICENSE)
