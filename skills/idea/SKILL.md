# /idea — Capture & Evaluate an Idea
 
Capture and evaluate the following idea: **$ARGUMENTS**
 
## Instructions
 
This is Stage 1 of the workflow — raw ideation. Help structure and stress-test the idea before committing time to a full PRD.
 
---
 
**Title**: [Project / New?] Idea — [Short Name]
**Date**: [today]
**Status**: Raw Idea
 
---
 
## The Idea
Restate the idea clearly in 2–3 sentences. Make it concrete.
 
## Problem It Solves
- Who has this problem?
- How painful is it? (1–5)
- How often does it occur?
 
## Proposed Solution
High-level: what does the user see/do? Not implementation yet.
 
## Why This, Why Now?
What triggered this idea? Is there urgency or is it nice-to-have?
 
## Rough Value Assessment
- **User impact**: High / Medium / Low
- **Effort estimate**: XS / S / M / L / XL
- **Priority signal**: Must-have / Should-have / Nice-to-have / Backlog
 
## Risks & Unknowns
What would need to be true for this idea to work well?
What could make it fail or be harder than expected?
 
## Similar / Related
- Existing features this relates to or could conflict with
- Similar patterns in other apps worth looking at
 
## Recommendation
**Go → Write PRD** | **Explore more** | **Park for later** | **Drop**
 
Reason: ...
 
## Next Step (if Go)
→ Use `/prd [feature name]` to write the full requirements

---

## Save to Notion

After generating the idea evaluation above, save it to the project's Notion space.

### Instructions

1. **Ask the user** for:
   - **Project name** — which project is this idea for (e.g., Task Tracker, Expense Tracker, Personal AI)
   - **Target version** — the version this idea targets (e.g., `1.2.0`)

2. **Find the project in Notion** — use `mcp__notion__notion-search` to locate the project's root page.

3. **Find the version page** — use `mcp__notion__notion-fetch` on the project page to list its children, then look for a child page matching the target version number (e.g., `1.2.0`).

4. **If the version page does not exist** — create it using `mcp__notion__notion-create-pages` as a child of the project page, with the title set to the version number (e.g., `1.2.0`).

5. **Create the idea page** — use `mcp__notion__notion-create-pages` as a child of the version page:
   - **Title**: `[Project] Idea — [Short Name]` (follow the Notion documentation standard)
   - **Icon**: 💡
   - **Content**: The full idea evaluation output above, formatted in Notion Markdown:
     - Include all sections: The Idea, Problem It Solves, Proposed Solution, Why This Why Now, Rough Value Assessment, Risks & Unknowns, Similar / Related, Recommendation, Next Step
     - Add metadata at the top: **Date**: today, **Status**: Raw Idea

6. **Confirm to the user** — share the Notion page URL once created.
