# Dispatch Prompt

Route this triaged issue to the appropriate project.

## Process

1. **Validate Target**: Check if `projects/<target>/` exists
2. **Check Relevance**: Confirm the issue aligns with target project scope
3. **Create Reference**: Add a comment to the issue with dispatch details
4. **Update Labels**: Add `status/dispatched` label
5. **Optional Close**: Close with reference if fully dispatched

## Dispatch Comment Template

```markdown
## Dispatched to: `<project-name>`

This issue has been routed to the **<project-name>** project.

**Relevance**: [Why this belongs in the target project]

**Action Items**:
- [ ] Review in context of <project-name>
- [ ] Create specific tasks if needed
- [ ] Update project roadmap

[Link to project: projects/<project-name>/]
```

## For External Repos

If dispatching to a repo outside this repository:
1. Do NOT attempt to create issues in external repos
2. Add a comment with manual dispatch instructions:
   ```markdown
   ## External Dispatch Suggested: `<repo-name>`

   This issue should be created in an external repository.

   **Target**: [repo URL or name]
   **Suggested Title**: [title]
   **Context**: [summary of issue]

   Please manually create the issue and link back here.
   ```
3. Apply label `status/pending-external`
