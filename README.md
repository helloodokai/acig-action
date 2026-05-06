# ACIG Action

GitHub Action for [ACIG](https://github.com/helloodokai/acig) — tiered code review via cheap critics + frontier adjudication.

## Quick Start

```yaml
name: acig

on:
  pull_request:
    types: [opened, synchronize, reopened]

concurrency:
  group: acig-${{ github.event.pull_request.number }}
  cancel-in-progress: true

permissions:
  contents: read
  pull-requests: write
  checks: write

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get GitHub App token
        id: app-token
        uses: actions/create-github-app-token@v2
        if: ${{ secrets.ACIG_APP_ID != '' && secrets.ACIG_APP_PRIVATE_KEY != '' }}
        with:
          app-id: ${{ secrets.ACIG_APP_ID }}
          private-key: ${{ secrets.ACIG_APP_PRIVATE_KEY }}

      - uses: helloodokai/acig-action@v1
        with:
          github-token: ${{ steps.app-token.outputs.token || secrets.GITHUB_TOKEN }}
          ollama-api-key: ${{ secrets.OLLAMA_API_KEY }}
```

## With GitHub App (ACIG branding)

Reviews appear as "ACIG" with your custom logo instead of `github-actions[bot]`. One-time setup:

1. Create a GitHub App at [settings/apps/new](https://github.com/settings/apps/new) with:
   - **Permissions**: Pull requests: Read & write, Checks: Read & write, Contents: Read-only
   - **Events**: Pull request, Pull request review, Check run, Check suite
   - Leave webhook URL empty
2. Generate a private key in the app settings
3. Install the app on your repo
4. Add secrets: `ACIG_APP_ID` and `ACIG_APP_PRIVATE_KEY` (or run `acig setup-app`)

## Without GitHub App (default)

```yaml
- uses: helloodokai/acig-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    ollama-api-key: ${{ secrets.OLLAMA_API_KEY }}
```

Reviews appear as `github-actions[bot]`.

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `github-token` | (required) | Token for posting reviews. Use app token for ACIG branding or `GITHUB_TOKEN` for default. |
| `profile` | `cloud` | Model profile: `cloud` or `local` |
| `version` | `1.5.2` | acig version to install |
| `format` | `both` | Output format: `json`, `md`, or `both` |
| `budget` | (from config) | Per-run budget in USD |
| `blocking` | `true` | Fail the build on warn/block verdicts. Set to `false` for non-blocking mode. |
| `ollama-api-key` | | Ollama Cloud API key |
| `openai-api-key` | | OpenAI API key (optional) |
| `anthropic-api-key` | | Anthropic API key (optional) |

## Required secrets

| Secret | Description |
|--------|-------------|
| `OLLAMA_API_KEY` | **Required.** Get one at [ollama.com/settings/keys](https://ollama.com/settings/keys) |
| `ACIG_APP_ID` | Optional. GitHub App ID for ACIG branding |
| `ACIG_APP_PRIVATE_KEY` | Optional. Private key for GitHub App authentication |

## Blocking behavior

- `blocking: true` (default) — `warn` and `block` verdicts fail the build (exit 1)
- `blocking: false` — all verdicts pass (exit 0), findings still posted as reviews