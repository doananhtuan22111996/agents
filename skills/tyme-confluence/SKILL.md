# /tyme-confluence — Write to Confluence

Create or update a Confluence page for: **$ARGUMENTS**
Usage: `/tyme-confluence ONC-123 implementation notes` or `/tyme-confluence sprint-33 retrospective`

## Instructions

Use the Confluence MCP to create or update documentation. All Tyme technical docs go to Confluence, not Notion.

---

## Step 1 — Find the Right Space / Parent Page
```
getConfluenceSpaces()
getPagesInConfluenceSpace(spaceKey: "ENG")
```

## Step 2 — Determine Action
```
searchConfluenceUsingCql(cql: "title = \"ONC-XXX Implementation\" AND space = \"ENG\"")
getConfluencePage(pageId: "XXXXX")
```

## Step 3 — Page Templates

### Implementation Notes
```
Title: ONC-XXX — [Ticket Title] — Implementation Notes

## Summary / ## Ticket / ## Approach / ## Key Changes / ## Known Limitations / ## PR
```

### Sprint Retrospective
```
Title: Sprint [N] — Mobile Retrospective

## What went well / ## What didn't go well / ## Action items
```

### Technical Decision
```
Title: [Topic] — Technical Decision

## Context / ## Options Considered / ## Decision / ## Rationale / ## Consequences
```

## Step 4 — Create or Update
```
createConfluencePage(spaceKey: "ENG", title: "...", content: "...", parentPageId: "XXXXX")
updateConfluencePage(pageId: "XXXXX", title: "...", content: "...", version: N)
```

## Step 5 — Link Back to Jira
```
addCommentToJiraIssue(issueKey: "ONC-XXX", comment: "Confluence doc: [page URL]")
```
