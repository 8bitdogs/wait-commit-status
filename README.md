# wait-commit-status

A GitHub composite action that polls a commit status context until it reaches
the expected state or a timeout is exceeded.

## Usage

```yaml
- uses: 8bitdogs/wait-commit-status@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    context: 'ci/my-check'
    state: 'success'
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `token` | ✅ | — | GitHub token used to authenticate API requests |
| `repository` | | `${{ github.repository }}` | Repository in `owner/repo` format |
| `sha` | | `${{ github.sha }}` | Commit SHA to watch |
| `state` | | `success` | Expected state to wait for: `error`, `failure`, `pending`, or `success` |
| `not-found-state` | | `pending` | State to use when the specified context does not exist yet: `success`, `error`, or `pending` |
| `context` | ✅ | — | Status context name (e.g. `ci/circleci: build`) |
| `fail-states` | | `error,failure` | Comma-separated states that should fail immediately |
| `pending-states` | | `pending` | Comma-separated states that should continue polling |
| `interval` | | `5` | Polling interval in seconds |
| `timeout` | | `600` | Maximum seconds to wait before failing |

Notes:

- If the target context has not been created yet, the action maps it to `not-found-state` (default `pending`).
- The action fails fast when an observed state matches any value in `fail-states` (default `error,failure`).
- The action keeps polling only while state is in `pending-states` unless expected state is reached.

## Outputs

| Output | Description |
|--------|-------------|
| `state` | The final observed state of the commit status |

## Example — wait for an external CI status before deploying

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Wait for external CI to pass
        uses: 8bitdogs/wait-commit-status@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          context: 'ci/circleci: build'
          state: 'success'
          not-found-state: 'pending'
          fail-states: 'error,failure'
          pending-states: 'pending'
          interval: '5'
          timeout: '300'

      - name: Deploy
        run: ./deploy.sh
```

## Example — watch a different repository / SHA

```yaml
- uses: 8bitdogs/wait-commit-status@main
  with:
    token: ${{ secrets.PAT }}
    repository: my-org/other-repo
    sha: ${{ steps.get-sha.outputs.sha }}
    context: 'security/scan'
    state: 'success'
    timeout: '120'
```
