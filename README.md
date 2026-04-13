<p align="center"><img src="assets/decodie-logo.png" alt="Decodie" width="200"></p>

# Decodie Bot

An interactive GitHub bot that responds to `@decodie-bot` mentions in pull request comments. Mention it on highlighted code to get an explanation, or in a top-level comment to generate [Decodie](https://decodie.owenbush.dev) learning entries for the PR.

Built on top of [claude-code-action](https://github.com/anthropics/claude-code-action) — runs the official [Decodie skill](https://github.com/owenbush/decodie-skill) (`/decodie:explain` and `/decodie:analyze`) on demand from PR conversations.

## How It Works

The bot responds to two types of PR comments:

### Inline review comments (code explanations)

When you highlight lines of code in a PR review and mention `@decodie-bot`, the bot runs `/decodie:explain` on the highlighted code and replies with:

- **Summary** — what the code does at a high level
- **Detailed breakdowns** — non-obvious sections explained with patterns identified
- **Potential issues** — bugs, security concerns, edge cases with severity levels
- **Improvements** — refactoring opportunities and modern alternatives
- **Key concepts** — transferable patterns and principles

This is ephemeral — nothing is committed to the repo.

### Top-level PR comments (entry generation)

When you mention `@decodie-bot` in a regular PR comment, the bot runs `/decodie:analyze` on the PR's changed files and generates structured Decodie learning entries. These can optionally be committed to `.decodie/` in the PR branch.

## Quick Start

```yaml
name: Decodie Bot
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  bot:
    if: contains(github.event.comment.body, '@decodie-bot')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      issues: write
    steps:
      - uses: actions/checkout@v4
      - uses: owenbush/decodie-github-bot@v1
        with:
          # Use ONE of:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          # claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

## Usage Examples

### Explain highlighted code

In a PR review, select some lines of code, start a review comment, and write:

> `@decodie-bot explain this`

or simply:

> `@decodie-bot`

The bot replies in the review thread with a detailed explanation.

### Analyze the whole PR

In a top-level PR comment, write:

> `@decodie-bot analyze this PR`

or simply:

> `@decodie-bot`

The bot analyzes changed files and replies with a summary of the generated entries.

### Analyze a specific file

> `@decodie-bot analyze src/auth/middleware.ts`

### Explain a specific file from a top-level comment

> `@decodie-bot explain src/auth/middleware.ts`

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `anthropic-api-key` | Anthropic API key (provide this or `claude-code-oauth-token`) | `""` |
| `claude-code-oauth-token` | Claude Code OAuth token (provide this or `anthropic-api-key`) | `""` |
| `trigger-phrase` | The mention phrase that triggers the bot | `@decodie-bot` |
| `model` | Claude model to use | `claude-sonnet-4-6` |
| `mode` | Analysis mode for `/decodie:analyze`: `selective` or `exhaustive` | `selective` |
| `max-files` | Maximum files to analyze (0 for no limit) | `10` |
| `include` | Comma-separated glob patterns for files to include in analysis | `""` |
| `exclude` | Comma-separated glob patterns for files to exclude from analysis | `""` |
| `commit` | Whether to commit generated entries from `/decodie:analyze` to `.decodie/` | `false` |
| `github-token` | GitHub token with repo and pull request permissions | `${{ github.token }}` |

### Authentication

Provide **one** of:
- **`anthropic-api-key`** — pay-per-token from [console.anthropic.com](https://console.anthropic.com/)
- **`claude-code-oauth-token`** — uses your Claude Pro/Max subscription (run `claude setup-token` to generate one)

Store whichever you use as a GitHub repository secret.

## Permissions

The workflow needs these permissions:

```yaml
permissions:
  contents: read        # To read repository files
  pull-requests: write  # To post PR comments
  issues: write         # To respond to issue comments on PRs
```

If you enable `commit: 'true'` for analyze results, change `contents` to `write` and use `actions/checkout` with `ref: ${{ github.head_ref }}`:

```yaml
permissions:
  contents: write
  pull-requests: write
  issues: write

steps:
  - uses: actions/checkout@v4
    with:
      ref: ${{ github.head_ref }}
  - uses: owenbush/decodie-github-bot@v1
    with:
      anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
      commit: 'true'
```

## Cost Control

The bot runs Claude on every matching comment, so costs scale with usage. To keep costs predictable:

- Only users with **write permissions** to the repository can trigger the bot (enforced by `claude-code-action`)
- Use `max-files` to cap the number of files analyzed per invocation (default: 10)
- Use `include`/`exclude` to limit analysis scope
- `selective` mode (the default) generates 3-5 entries per file
- Inline explain requests are typically cheap — they operate on a single code selection

## Decodie Bot vs Decodie GitHub Action

| | [Decodie Bot](https://github.com/owenbush/decodie-github-bot) | [Decodie GitHub Action](https://github.com/owenbush/decodie-github-action) |
|---|---|---|
| **Trigger** | On demand, via `@decodie-bot` mention | Automatic, on every PR open/update |
| **Commands** | `/decodie:explain` and `/decodie:analyze` | `/decodie:analyze` only |
| **Output** | Reply comment in the PR thread | PR comment and/or committed entries |
| **Commits to `.decodie/`** | Off by default (opt-in) | On by default (opt-out) |
| **Cost model** | Per invocation (user-triggered) | Per PR (automatic) |
| **Best for** | Interactive code review, on-demand explanations | Automatic documentation as part of CI |

You can use both together — the action documents every PR automatically, and the bot answers specific questions during review.

## The Decodie Ecosystem

| Project | Description |
|---------|-------------|
| [decodie-skill](https://github.com/owenbush/decodie-skill) | Claude Code skill for real-time and retroactive code documentation |
| [decodie-ui](https://github.com/owenbush/decodie-ui) | Web-based browser with lessons, progress tracking, and Q&A |
| [decodie-vscode](https://marketplace.visualstudio.com/items?itemName=owenbush.decodie-vscode) | VSCode extension for browsing entries in your editor |
| [decodie-github-action](https://github.com/owenbush/decodie-github-action) | GitHub Action for automatic PR analysis |
| [decodie-ddev](https://github.com/owenbush/decodie-ddev) | DDEV add-on for local development |
| [decodie-core](https://github.com/owenbush/decodie-core) | Shared data layer (types, parser, reference resolver) |
| [decodie.owenbush.dev](https://decodie.owenbush.dev) | Project homepage |

## License

MIT
