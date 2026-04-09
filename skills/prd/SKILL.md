# /prd — Product Requirements Document
 
Write a complete PRD for the following feature: **$ARGUMENTS**
 
## Instructions
 
Produce a structured PRD ready to paste into Notion. Follow this exact structure:
 
---
 
**Title**: [Project] PRD — [Feature Name]
**Status**: Draft
**Created**: [today's date]
**Author**: Solo
 
---
 
## 1. Overview
- **Problem**: What problem does this solve for the user?
- **Goal**: What outcome do we want?
- **Success metric**: How do we know it's working? (measurable)
 
## 2. Background & Context
- Why now? What triggered this idea?
- Any related features or dependencies?
 
## 3. User Stories
List as: `As a [user], I want to [action] so that [value].`
Cover happy path + key edge cases.
 
## 4. Functional Requirements
Number each requirement. Be specific and testable.
- FR-01: ...
- FR-02: ...
 
## 5. Non-Functional Requirements
- Performance: e.g., "loads within 300ms on mid-range device"
- Offline: e.g., "works without network, syncs when reconnected"
- Security: e.g., "data encrypted at rest using EncryptedSharedPreferences"
- Accessibility: e.g., "supports TalkBack"
 
## 6. Out of Scope (v1)
Explicitly list what is NOT included in this version. Prevents scope creep.
 
## 7. Open Questions
Unresolved decisions or assumptions that need validation.
 
## 8. Acceptance Criteria
Checklist format. Each item should be verifiable by a tester.
- [ ] ...
- [ ] ...
 
---
 
After writing the PRD, suggest 2–3 open questions that would most impact the design decisions, if any are not already covered.

---

## Save to Notion

After generating the PRD above, save it to the project's Notion space.

### Instructions

1. **Ask the user** for:
   - **Project name** — which project is this PRD for (e.g., Task Tracker, Expense Tracker, Personal AI)
   - **Target version** — the version this PRD targets (e.g., `1.2.0`)

2. **Find the project in Notion** — use `mcp__notion__notion-search` to locate the project's root page.

3. **Find the version page** — use `mcp__notion__notion-fetch` on the project page to list its children, then look for a child page matching the target version number (e.g., `1.2.0`).

4. **If the version page does not exist** — create it using `mcp__notion__notion-create-pages` as a child of the project page, with the title set to the version number (e.g., `1.2.0`).

5. **Create the PRD page** — use `mcp__notion__notion-create-pages` as a child of the version page:
   - **Title**: `[Project] PRD — [Feature Name]` (follow the Notion documentation standard)
   - **Icon**: 📋
   - **Content**: The full PRD output above, formatted in Notion Markdown:
     - Include all sections: Overview, Background & Context, User Stories, Functional Requirements, Non-Functional Requirements, Out of Scope, Open Questions, Acceptance Criteria
     - Add metadata at the top: **Status**: Draft, **Created**: today, **Author**: Solo

6. **Confirm to the user** — share the Notion page URL once created.
