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
| `context` | ✅ | — | Status context name (e.g. `ci/circleci: build`) |
| `interval` | | `5` | Polling interval in seconds |
| `timeout` | | `600` | Maximum seconds to wait before failing |

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
