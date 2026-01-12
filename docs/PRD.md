# Triage System - Product Requirements Document

## Executive Summary

A GitHub-native idea capture and triage system that uses GitHub Issues as input and Claude Code Action (via GitHub Actions) to process, categorize, prioritize, and dispatch ideas to appropriate projects.

**Target User**: Developer who wants to capture ideas on-the-go (mobile GitHub app) and have them automatically triaged and routed to the right project.

## Problem Statement

When ideas strike while away from your development environment, you need a frictionless way to:
1. Capture the idea quickly
2. Have it analyzed and categorized
3. Route it to the appropriate project
4. Convert it to actionable work items

Current solutions require manual triage, context switching, or complex tooling.

## Solution

Use GitHub Issues + Claude Code Action to create an intelligent triage pipeline:

```
[Mobile/Web] → [GitHub Issue] → [Claude Triage] → [Labeled/Dispatched Issue or Todo]
```

## Why GitHub Issues (Not PRs)

| Factor | Issues | Pull Requests |
|--------|--------|---------------|
| Mobile creation | ✅ Native support | ❌ Requires branch/code |
| Idea capture | ✅ Designed for this | ❌ Code-centric |
| Lightweight | ✅ No repo changes | ❌ Creates commits |
| Labels/Projects | ✅ Full support | ✅ Full support |
| Templates | ✅ Issue templates | ✅ PR templates |

**Decision**: Issues are the correct primitive for idea capture.

## Core Features

### Phase 1: Basic Triage (MVP)

#### 1.1 Issue Creation
- User creates issue in `Triage` repo with title and description
- Optional: use issue template for structured input

#### 1.2 Manual Triage Trigger
- User comments `@claude /triage` on any issue
- GitHub Action triggers on `issue_comment` event
- Claude analyzes issue and responds with:
  - **Category**: feature, bug, research, documentation, refactor
  - **Priority**: P0 (critical) → P3 (backlog)
  - **Suggested Labels**: Applied automatically
  - **Summary**: One-line distillation
  - **Next Steps**: Recommended actions

#### 1.3 Label Application
Claude applies labels via `gh issue edit`:
- `type/feature`, `type/bug`, `type/research`, etc.
- `priority/p0` through `priority/p3`
- `status/triaged`

### Phase 2: Slash Commands

#### 2.1 Command: `/categorize`
```
@claude /categorize
```
- Analyzes issue content
- Suggests and applies type labels
- Explains reasoning

#### 2.2 Command: `/prioritize`
```
@claude /prioritize
```
- Evaluates urgency, impact, effort
- Applies priority label
- Provides rationale

#### 2.3 Command: `/dispatch <target>`
```
@claude /dispatch datacenter-feasibility
```
- Creates linked issue in target project (if in `projects/`)
- Or creates reference comment if external repo
- Adds `status/dispatched` label
- Closes original with link

#### 2.4 Command: `/todo`
```
@claude /todo
```
- Breaks idea into actionable tasks
- Creates checklist in issue body or separate issues
- Useful for complex features

### Phase 3: Automation & Intelligence

#### 3.1 Auto-Triage on Issue Creation
```yaml
on:
  issues:
    types: [opened]
```
- Automatically runs `/triage` on new issues
- No manual trigger needed

#### 3.2 Duplicate Detection
- Search existing issues for similar content
- Link potential duplicates
- Suggest consolidation

#### 3.3 Cross-Project Context
- Claude reads `CLAUDE.md` from target projects
- Provides context-aware suggestions
- Understands project-specific terminology

### Phase 4: Sub-Project Management

#### 4.1 Project Incubation
```
projects/
├── datacenter-feasibility/
├── grid-monitor/
└── site-selector/
```
- New project ideas start in `projects/`
- Full development happens in subdirectory
- Spin out when mature

#### 4.2 Project Spin-Out
```
@claude /spinout <project-name>
```
- Creates new GitHub repository
- Migrates code via subtree split
- Updates references
- Archives `projects/<name>/`

## Technical Architecture

### GitHub Actions Workflow

```yaml
# .github/workflows/triage-issue.yml
name: Triage Issue

on:
  issue_comment:
    types: [created, edited]

jobs:
  triage:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            REPO: ${{ github.repository }}
            ISSUE NUMBER: ${{ github.event.issue.number }}
            ISSUE TITLE: ${{ github.event.issue.title }}
            ISSUE BODY: ${{ github.event.issue.body }}
            COMMAND: ${{ github.event.comment.body }}

            Process the triage command. Available commands:
            - /triage: Full analysis with category, priority, summary, next steps
            - /categorize: Determine and apply type label
            - /prioritize: Determine and apply priority label
            - /dispatch <project>: Link/move to target project
            - /todo: Break into actionable items

            Use gh CLI to apply labels and update issues.
          claude_args: |
            --allowedTools "Bash(gh issue:*),Bash(gh search:*),Read"
```

### Prompt Templates

Store reusable prompts in `prompts/`:

```markdown
<!-- prompts/triage.md -->
# Triage Analysis

Analyze this issue and provide:

## Category
Determine the type: feature, bug, research, documentation, refactor, question

## Priority
- P0: Critical - blocking, security issue, production down
- P1: High - important feature, significant bug
- P2: Medium - nice to have, minor bug
- P3: Low - backlog, future consideration

## Summary
One sentence capturing the core idea.

## Next Steps
2-3 concrete actions to move this forward.

## Labels
Apply using: gh issue edit <number> --add-label "type/<category>,priority/<p#>,status/triaged"
```

### Authentication (Claude Max OAuth)

The workflow uses:
- `CLAUDE_CODE_OAUTH_TOKEN` - Generated via `claude setup-token`, stored in repository secrets
- `GITHUB_TOKEN` - Auto-provided, used for `gh` CLI operations

**Setup steps:**
1. Run `/install-github-app` in Claude Code CLI
2. Run `claude setup-token` to generate OAuth token
3. Add token to repository Settings → Secrets → `CLAUDE_CODE_OAUTH_TOKEN`

**Note:** OAuth tokens expire (~1 hour). This is a [known issue](https://github.com/anthropics/claude-code/issues/11016). May need periodic regeneration.

## User Flows

### Flow 1: Quick Idea Capture (Mobile)

1. Open GitHub mobile app
2. Navigate to `Triage` repo
3. Create new issue with idea
4. Comment `@claude /triage`
5. Receive categorized, prioritized response
6. Later: review and dispatch to appropriate project

### Flow 2: Structured Feature Request

1. Create issue using "Feature Request" template
2. Fill in: Problem, Proposed Solution, Alternatives
3. Submit → Auto-triage runs
4. Review Claude's analysis
5. Comment `@claude /dispatch site-selector` to route

### Flow 3: Research Task

1. Create issue: "Research: European grid interconnection APIs"
2. Comment `@claude /triage`
3. Claude categorizes as `type/research`, suggests sources
4. Comment `@claude /todo` to break into research steps

## Success Metrics

| Metric | Target |
|--------|--------|
| Time from idea to triaged | < 2 minutes |
| Manual intervention rate | < 20% of issues |
| Correct categorization | > 90% accuracy |
| Mobile-friendly creation | 100% |

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| API costs | Medium | Use claude-3-haiku for simple commands |
| Rate limiting | Low | Batch operations, debounce triggers |
| Misclassification | Medium | Human review before dispatch |
| Workflow failures | Low | Error handling, retry logic |

## Implementation Roadmap

### Sprint 1: MVP Triage
- [ ] Create repository structure
- [ ] Implement `triage-issue.yml` workflow
- [ ] Create basic prompt templates
- [ ] Test `/triage` command

### Sprint 2: Full Commands
- [ ] Implement `/categorize`, `/prioritize`
- [ ] Implement `/dispatch` with internal projects
- [ ] Add `/todo` command
- [ ] Create issue templates

### Sprint 3: Automation
- [ ] Auto-triage on issue creation
- [ ] Duplicate detection
- [ ] Cross-project context reading

### Sprint 4: Polish
- [ ] Documentation
- [ ] Error handling
- [ ] Cost optimization
- [ ] External repo dispatch

## Open Questions

1. **Label taxonomy**: Should we use hierarchical labels (`type/feature`) or flat (`feature`)?
   - **Decision**: Hierarchical for clarity and filtering

2. **Multi-repo dispatch**: How to handle external repos?
   - **Approach**: Create reference issue/comment, user manually creates in target

3. **Cost management**: How to prevent runaway API costs?
   - **Approach**: Use haiku for simple commands, sonnet for complex triage

---

## References

- [Claude Code Action](https://github.com/anthropics/claude-code-action) - Official GitHub Action
- [Solutions Guide](https://github.com/anthropics/claude-code-action/blob/main/docs/solutions.md) - Workflow patterns
- [GitHub Actions Triggers](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
