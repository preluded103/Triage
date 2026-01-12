# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

**Triage** is a GitHub Actions-powered idea triage system that lets you capture, categorize, and dispatch ideas to other projects from anywhere via GitHub Issues. When you create an issue with specific commands, Claude Code Action processes it and executes triage actions.

## Architecture

```
Triage/
├── .github/
│   └── workflows/
│       ├── triage-issue.yml      # Main triage workflow (issue_comment trigger)
│       ├── triage-dispatch.yml   # Manual dispatch workflow
│       └── auto-label.yml        # Auto-labeling on new issues
├── projects/                     # Sub-projects incubated here
│   └── <project-name>/           # Each sub-project has its own directory
├── prompts/                      # Reusable Claude prompts for triage actions
│   ├── categorize.md
│   ├── prioritize.md
│   └── dispatch.md
├── docs/
│   └── PRD.md                    # Product Requirements Document
└── CLAUDE.md
```

## How It Works

1. **Create Issue**: Open a GitHub Issue with your idea
2. **Triage Command**: Comment `@claude /triage` to trigger analysis
3. **Claude Processes**: GitHub Action invokes Claude Code Action
4. **Output**: Issue is labeled, prioritized, and optionally dispatched to target project

### Slash Commands

| Command | Description |
|---------|-------------|
| `@claude /triage` | Full triage: categorize, prioritize, suggest actions |
| `@claude /categorize` | Categorize issue type (feature, bug, research, etc.) |
| `@claude /prioritize` | Assign priority (P0-P3) with rationale |
| `@claude /dispatch <project>` | Move/link idea to a sub-project or external repo |
| `@claude /todo` | Convert issue to actionable todo items |

## Development

### Prerequisites

- Node.js 22 LTS
- pnpm (package manager)
- GitHub CLI (`gh`) authenticated

### Local Testing

```bash
# Validate workflow syntax
gh workflow list
gh workflow view triage-issue

# Test with act (local GitHub Actions runner)
act issue_comment -e test/events/issue_comment.json
```

### Authentication (Claude Max)

Uses OAuth token from Claude Max subscription (no API key needed):

1. Run `/install-github-app` in Claude Code CLI
2. Run `claude setup-token` to generate OAuth token
3. Add `CLAUDE_CODE_OAUTH_TOKEN` to repository Settings → Secrets

Note: `GITHUB_TOKEN` is automatically provided by GitHub Actions.

**Known limitation**: OAuth tokens expire (~1 hour). May need to regenerate via `claude setup-token` if workflows fail.

### Workflow Permissions

All workflows require these permissions in the YAML:

```yaml
permissions:
  contents: read
  issues: write
  pull-requests: write
```

## Creating Sub-Projects

Sub-projects live in `projects/<name>/` until ready to spin out:

```bash
mkdir -p projects/my-new-project
cd projects/my-new-project
# Initialize with appropriate structure
```

When ready to spin out:
1. Create new repository
2. Use `git subtree split` or simply copy
3. Update any cross-references in parent Triage repo

## Key Files

- `.github/workflows/triage-issue.yml` - Main entry point for all triage actions
- `prompts/*.md` - Claude prompt templates (edit these to customize behavior)
- `docs/PRD.md` - Full product requirements and roadmap

## Conventions

- **Issues**: Use for ideas, bugs, and feature requests (not PRs)
- **Labels**: Auto-applied by triage (type/*, priority/*, status/*)
- **Branches**: Not typically needed (this is a workflow/config repo)
- **Commits**: Conventional commits (`feat:`, `fix:`, `docs:`)
