# Tyme — Project Context

## Company & Team
- **Company**: Tyme
- **Environment**: Agile (Scrum sprints)
- **Role**: Senior Mobile Engineer (Android + iOS)
- **Work mode**: Team — follow team conventions strictly

## Tools
- **Version Control**: Bitbucket
- **Issue Tracker**: Jira (project key: ONC)
- **Documentation**: Confluence + Notion
- **CI/CD**: Bitbucket Pipelines

## Branch Naming
```
feature/ONC-{ticketNumber}-short-description
fix/ONC-{ticketNumber}-short-description
chore/ONC-{ticketNumber}-short-description
```

## Commit Format
```
ONC-{ticketNumber}: <type>: short description
```
Example: `ONC-123: feat: add recurring payment screen`

## PR Format (Bitbucket)
- Title: `ONC-{ticketNumber}: <type>: description`
- Body: What / Why / How / Test plan / Jira link
- Always link Jira ticket in PR description

## Workflow (per feature/task)
```
1. Pick Jira ticket (ONC-XXX) → move to In Progress
2. Create branch: feature/ONC-XXX-description
3. Implement (Android or iOS)
4. Self-review: /tyme-review
5. Commit: ONC-XXX: feat: description
6. Push to Bitbucket
7. Create PR → /tyme-pr ONC-XXX
8. Update Jira ticket with PR link + notes
9. Wait for CI (Bitbucket Pipelines)
10. Team review → address comments
11. Merge to master/main
12. /tyme-jira ONC-XXX done
```

## Code Standards
- Follow existing code style — consistency over personal preference
- No force push to main/master/develop
- PR must have at least 1 approval before merge
- CI must be green before merge
