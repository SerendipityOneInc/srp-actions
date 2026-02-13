# srp-actions

Reusable GitHub Actions workflows for SerendipityOne repositories.

## Workflows

### Claude Auto Review

Automatic PR review powered by Claude. Optionally produces a pass/fail verdict.

**Caller example** (`.github/workflows/auto-review.yml`):

```yaml
name: Auto Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  claude-review:
    uses: SerendipityOneInc/srp-actions/.github/workflows/claude-review.yaml@main
    # with:
    #   model: "us.anthropic.claude-sonnet-4-20250514-v1:0"
    #   timeout_minutes: "60"
    #   allowed_bots: "srp-claude-assistant"
    #   enable_verdict: true
    #   additional_prompt: "Pay special attention to security."
    secrets: inherit
```

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | string | `us.anthropic.claude-sonnet-4-20250514-v1:0` | Bedrock model ID |
| `timeout_minutes` | string | `60` | Action timeout |
| `allowed_bots` | string | `""` | Bot usernames allowed to trigger |
| `enable_verdict` | boolean | `true` | Append verdict prompt and parse APPROVE/REQUEST_CHANGES |
| `additional_prompt` | string | `""` | Extra instructions appended to the review prompt |

---

### Claude Assistant

Interactive `@claude` mention handler for PRs and issues.

**Caller example** (`.github/workflows/claude-assistant.yml`):

```yaml
name: Claude Assistant

on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
  pull_request_review:
    types: [submitted]
  issues:
    types: [opened, assigned]

jobs:
  claude-assistant:
    if: |
      (github.event_name == 'issue_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review_comment' && contains(github.event.comment.body, '@claude')) ||
      (github.event_name == 'pull_request_review' && contains(github.event.review.body, '@claude')) ||
      (github.event_name == 'issues' && contains(github.event.issue.body, '@claude'))
    uses: SerendipityOneInc/srp-actions/.github/workflows/claude-assistant.yaml@main
    # with:
    #   model: "us.anthropic.claude-sonnet-4-20250514-v1:0"
    #   timeout_minutes: "60"
    #   allowed_bots: "srp-claude-assistant"
    secrets: inherit
```

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `model` | string | `us.anthropic.claude-sonnet-4-20250514-v1:0` | Bedrock model ID |
| `timeout_minutes` | string | `60` | Action timeout |
| `allowed_bots` | string | `""` | Bot usernames allowed to trigger |

> **Note:** The `on:` triggers and `@claude` filtering must live in the caller — `workflow_call` cannot forward event subscriptions.

---

### Python Code Quality

Ruff linting, Pyright type checking, and pytest with optional coverage enforcement.

**Caller example** (`.github/workflows/code-quality.yml`):

```yaml
name: Code Quality

on:
  pull_request:
    types: [opened, synchronize]
  push:
    branches: [main, develop]

jobs:
  code-quality:
    uses: SerendipityOneInc/srp-actions/.github/workflows/python-code-quality-v2.yml@main
    with:
      python_version: '3.10'
      repo_name: ${{ github.event.repository.name }}
      # ruff_config_file: 'pyproject.toml'
      # pyright_config_file: 'pyrightconfig.json'
      # enable_coverage: true
      # coverage_threshold: '80'
      # coverage_source: 'app'
    secrets:
      FEISHU_CUSTOMERBOT_WEBHOOK: ${{ secrets.FEISHU_CUSTOMERBOT_WEBHOOK }}
      FEISHU_CUSTOMERBOT_SECRET: ${{ secrets.FEISHU_CUSTOMERBOT_SECRET }}
      GH_RELEASE_TOKEN: ${{ secrets.GH_RELEASE_TOKEN }}
```

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `python_version` | string | `3.10` | Python version |
| `repo_name` | string | *(required)* | Repository name |
| `runner_name` | string | `ubuntu-latest` | GitHub runner to use |
| `ruff_config_file` | string | `""` | Path to ruff config |
| `pyright_config_file` | string | `""` | Path to pyright config |
| `enable_coverage` | boolean | `false` | Enable coverage reporting |
| `coverage_threshold` | string | `80` | Minimum coverage percentage |
| `coverage_source` | string | `app` | Source directory for coverage |

---

## Required Secrets

All Claude workflows require:

| Secret | Description |
|--------|-------------|
| `APP_ID` | GitHub App ID for token generation |
| `APP_PRIVATE_KEY` | GitHub App private key |
| `AWS_ROLE_TO_ASSUME` | AWS IAM role ARN for Bedrock access (OIDC) |

Python Code Quality requires:

| Secret | Description |
|--------|-------------|
| `FEISHU_CUSTOMERBOT_WEBHOOK` | Feishu bot webhook URL |
| `FEISHU_CUSTOMERBOT_SECRET` | Feishu bot webhook secret |
| `GH_RELEASE_TOKEN` | GitHub PAT for accessing internal packages |

---

## Other Workflows

| Workflow | Description |
|----------|-------------|
| `build-and-push-image.yml` | Build and push Docker image to registry |
| `build-and-push-image-cached.yml` | Build and push with registry-based caching |
| `build-and-push-helm-chart.yml` | Package and push Helm chart |
| `build-and-push-pypi.yml` | Build and publish Python package |
| `deploy-to-gke.yml` | Deploy to Google Kubernetes Engine |
| `update-k8s-secrets.yml` | Update Kubernetes secrets |
| `cleanup-old-images.yml` | Remove old container images |
| `cleanup-old-tags.yml` | Remove old git tags |
