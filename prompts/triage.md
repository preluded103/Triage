# Triage Prompt

Analyze this issue and provide comprehensive triage.

## Category

Determine the type:
- **feature**: New functionality or enhancement
- **bug**: Something broken or not working as expected
- **research**: Investigation, learning, or exploration task
- **docs**: Documentation improvements
- **refactor**: Code improvement without behavior change

## Priority

Assess urgency and importance:
- **P0 (Critical)**: Security issue, production down, blocking work
- **P1 (High)**: Important feature, significant bug, time-sensitive
- **P2 (Medium)**: Nice to have, minor bug, improves quality of life
- **P3 (Low)**: Backlog item, future consideration, minor polish

Consider:
- Impact: How many users/projects affected?
- Urgency: Is there a deadline or blocking dependency?
- Effort: Quick win or major undertaking?

## Summary

Distill the issue into one clear sentence that captures the core request.

## Next Steps

Provide 2-3 concrete actions to move this forward:
1. Immediate action (what to do right now)
2. Follow-up action (what comes next)
3. Long-term consideration (optional)

## Labels

Apply using gh CLI:
```bash
gh issue edit <number> --add-label "type/<category>,priority/<p#>,status/triaged"
```
