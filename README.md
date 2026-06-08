# Cora AI Code Review

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Cora%20AI%20Code%20Review-blue?logo=github)](https://github.com/marketplace/actions/cora-ai-code-review)
[![cora-cli](https://img.shields.io/badge/cora--cli-v0.4.6-blue?logo=rust)](https://github.com/codecoradev/cora-cli)

AI-powered code review for pull requests. Reviews your diffs, posts a formatted PR comment with findings, and optionally uploads SARIF results to GitHub Code Scanning.

**BYOK** — bring your own LLM API key. Works with any OpenAI-compatible provider (OpenAI, Anthropic, Groq, Ollama, ZAI, etc.).

## Features

- 🔍 **Automated PR review** — reviews every PR diff for bugs, security issues, and best practices
- 💬 **PR comments** — posts a formatted comment with findings grouped by severity
- 📊 **SARIF upload** — integrates with GitHub Code Scanning
- 🔐 **BYOK** — use your own LLM API key, no vendor lock-in
- 🔁 **Retry logic** — resilient to transient network errors and CDN propagation delays
- ✅ **Checksum verification** — verifies binary integrity with SHA-256 checksums

## Quick Start

### 1. Add secrets

Go to **Settings → Secrets and variables → Actions** and add:

| Secret | Required | Description |
|--------|----------|-------------|
| `CORA_API_KEY` | ✅ Yes | Your LLM API key |
| `CORA_BASE_URL` | No | LLM API base URL (e.g., `https://api.openai.com/v1`) |
| `CORA_MODEL` | No | LLM model ID (e.g., `gpt-4o-mini`) |

### 2. Add workflow

Create `.github/workflows/cora-review.yml`:

```yaml
name: Cora AI Code Review

on:
  pull_request_target:
    branches: [main, develop]
    types: [opened, synchronize, ready_for_review, reopened]

concurrency:
  group: cora-review-${{ github.event.pull_request.number }}
  cancel-in-progress: true

permissions:
  contents: read
  pull-requests: write
  security-events: write

jobs:
  cora-review:
    name: Cora Review
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - name: Checkout PR head
        uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 0
          persist-credentials: false

      - name: Run Cora AI Code Review
        uses: codecoradev/cora-review-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          cora-api-key: ${{ secrets.CORA_API_KEY }}
          cora-base-url: ${{ secrets.CORA_BASE_URL }}
          cora-model: ${{ secrets.CORA_MODEL }}
```

That's it! Every PR will now get an AI code review comment.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `github-token` | ✅ Yes | — | GitHub token for PR comments and SARIF upload |
| `cora-api-key` | ✅ Yes | — | LLM API key |
| `cora-base-url` | No | — | LLM API base URL |
| `cora-model` | No | — | LLM model ID |
| `base-branch` | No | `origin/develop` | Base branch to compare against |
| `severity` | No | `major` | Minimum severity to report (`info`, `minor`, `major`, `critical`) |
| `cora-version` | No | `latest` | Pin a specific cora-cli version (e.g., `v0.4.6`) |
| `upload-sarif` | No | `true` | Upload SARIF to GitHub Code Scanning |

## Custom Configuration

Add a `.cora.yaml` to your project root to customize review behavior:

```yaml
# .cora.yaml
focus:
  - security
  - bugs
  - performance

rules:
  - No unwrap() in production code
  - All public functions must have error handling

ignore:
  files:
    - "vendor/**"
    - "**/*.generated.*"
  rules:
    - "style"
```

See [cora-cli configuration docs](https://github.com/codecoradev/cora-cli#configuration) for all options.

## How It Works

```
PR opened/synced
    │
    ▼
Checkout PR code
    │
    ▼
Download cora-cli binary (retry + checksum verified)
    │
    ▼
Run `cora review` on PR diff
    │
    ├── Post formatted PR comment with findings
    ├── Upload SARIF to Code Scanning (optional)
    └── Fail job if blocking issues found (severity ≥ threshold)
```

## Example PR Comment

> ## 🔍 Cora AI Code Review
>
> ⚠ **Review recommended** — warnings found.
>
> ### 🟡 Warning (2)
>
> - `src/api/auth.ts:42` — Potential SQL injection in user query
> - `src/utils/parser.ts:15` — Unchecked null return from parse()
>
> ---
> _Review powered by [cora-cli](https://github.com/codecoradev/cora-cli) · BYOK · MIT_

## Providers

Works with any OpenAI-compatible API:

| Provider | `cora-base-url` | `cora-model` |
|----------|-----------------|--------------|
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| Anthropic | `https://api.anthropic.com/v1` | `claude-3-haiku` |
| Groq | `https://api.groq.com/openai/v1` | `llama-3.1-8b` |
| Ollama | `http://localhost:11434/v1` | `llama3.1` |
| ZAI | `https://api.z.ai/api/coding/paas/v4` | `glm-5.1` |

## License

This action is part of the [cora-cli](https://github.com/codecoradev/cora-cli) project, licensed under MIT.
