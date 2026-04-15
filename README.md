# Compozerr GitHub Action

Deploy your projects to [Compozerr](https://compozerr.com) directly from GitHub Actions. Compozerr is a deployment platform that gives you your own VM with full Docker Compose support -- no containers-as-a-service abstractions, just your stack running on a real server.

## Quick Start

Deploy on every push to `main`:

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/deploy@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
```

## Actions

This repository provides two actions:

### `compozerr/github-action/setup`

Installs the Compozerr CLI and authenticates. Use this when you need to run custom CLI commands beyond just deploying.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `token` | Yes | -- | Compozerr API token (`cmpz_` prefix). Generate one at **Settings > API Tokens**. |
| `version` | No | `latest` | CLI version to install. |

### `compozerr/github-action/deploy`

Deploys your project to Compozerr. Can optionally install the CLI for you (pass `token`), or you can use the `setup` action first.

#### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `token` | No | -- | Compozerr API token. Not required if the `setup` action was already used in an earlier step. |
| `environment` | No | -- | Environment name to deploy to. If omitted, deploys to the default production environment. |
| `wait` | No | `true` | Wait for the deployment to complete before finishing the step. |
| `working-directory` | No | `.` | Directory containing `compozerr.json`. |

#### Outputs

| Output | Description |
|--------|-------------|
| `deployment-id` | ID of the created deployment. |
| `status-url` | URL to check deployment status. |
| `status` | Final deployment status (only populated when `wait` is `true`). |

## Workflow Examples

### Deploy on push to main

The simplest setup -- deploy every time you push to your main branch:

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/deploy@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
```

### Staging and production pipeline

Deploy to staging first, then promote to production:

```yaml
name: Deploy Pipeline
on:
  push:
    branches: [main]

jobs:
  staging:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/deploy@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
          environment: staging

  production:
    runs-on: ubuntu-latest
    needs: staging
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/deploy@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
          environment: production
```

### Setup + custom CLI commands

Use the `setup` action when you need to run CLI commands beyond deploy:

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/setup@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
      - run: compozerr deploy --wait
      - run: compozerr logs --since 5m
```

### Manual deploy with workflow_dispatch

Trigger deployments manually from the GitHub Actions UI, with an environment selector:

```yaml
name: Manual Deploy
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        default: 'production'
        type: choice
        options:
          - staging
          - production

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/deploy@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
          environment: ${{ inputs.environment }}
```

### Deploy staging on PR, production on merge

Preview changes in staging when a pull request is opened, then deploy to production when it merges:

```yaml
name: Deploy
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: compozerr/github-action/deploy@v1
        with:
          token: ${{ secrets.COMPOZERR_API_TOKEN }}
          environment: ${{ github.event_name == 'pull_request' && 'staging' || 'production' }}
```

## Getting an API Token

1. Go to [compozerr.com](https://compozerr.com) and sign in.
2. Navigate to **Settings > API Tokens**.
3. Click **Create Token** and copy the token (it starts with `cmpz_`).
4. In your GitHub repository, go to **Settings > Secrets and variables > Actions**.
5. Click **New repository secret**, name it `COMPOZERR_API_TOKEN`, and paste your token.

## License

MIT
