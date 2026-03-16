# /doc — Notion Documentation
 
Format the following into Notion-ready documentation: **$ARGUMENTS**
 
## Instructions
 
Take the provided notes, decisions, context, or raw content and transform it into a clean, structured Notion document. Use the standard format for all personal project docs.
 
---
 
**Title**: [Project] [DocType] — [Topic]
*(DocType: PRD | Breakdown | Design | ADR | Test Plan | Decision | Learning | Retrospective | Meeting Note | Research)*
 
**Status**: Draft
**Created**: [today's date]
**Last Updated**: [today's date]
 
---
 
## TL;DR
2–3 sentences. What is this document? What decision or knowledge does it capture? Why does it matter?
 
## Background
What context does the reader need? Why was this written?
 
## [Main Content Section]
The actual content, structured with headers and sub-headers.
 
Use tables for comparisons:
| Option | Pros | Cons |
|--------|------|------|
|        |      |      |
 
Use numbered lists for steps or ordered decisions.
 
## Decisions Made
Explicitly call out every decision and the reason behind it:
- **Decision**: [What was decided]
  **Reason**: [Why this was chosen over alternatives]
 
## Action Items
Anything that needs to follow from this document:
- [ ] [Action] — Owner: Solo — Due: [date]
 
## Open Questions
Anything still unresolved that should be revisited:
- [ ] Question: ...
  Status: Open / Blocked on: ...
 
## References
Links to related Notion pages, GitHub PRs, ADRs, or external resources.
 
---
 
Format for easy skimming: headers, short paragraphs, tables where comparison helps. Avoid walls of text.
