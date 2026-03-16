# /tyme-jira — Update Jira Ticket

Update Jira ticket: **$ARGUMENTS**
Usage: `/tyme-jira ONC-123` or `/tyme-jira ONC-123 done`

## Instructions

Use the Atlassian MCP to interact with Jira directly.

---

## Common Actions

### Get ticket details
```
getJiraIssue(issueKey: "ONC-XXX")
```

### Add implementation comment (after PR raised)
```
addCommentToJiraIssue(
  issueKey: "ONC-XXX",
  comment: "..."
)
```

Comment template:
```
## Implementation

**Branch**: feature/ONC-XXX-description
**PR**: [Bitbucket PR URL]

**What was done**:
[2-3 sentences]

**Notes for reviewer**:
[Edge cases, decisions made]

**Test coverage**:
[Unit tests added, manual scenarios tested]
```

### Transition ticket status
```
getTransitionsForJiraIssue(issueKey: "ONC-XXX")
transitionJiraIssue(issueKey: "ONC-XXX", transitionId: "XXX")
```

Common transitions: To Do → In Progress → In Review → Done

### Log work
```
addWorklogToJiraIssue(issueKey: "ONC-XXX", timeSpent: "2h", comment: "...")
```

---

## Typical Flow After PR Merged

1. Add comment with PR link + implementation notes
2. Transition status → Done
3. Log work if tracking time
