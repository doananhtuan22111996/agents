# /roadmap — Roadmap & Release Planning
 
Plan the roadmap and release timeline for: **$ARGUMENTS**
 
## Instructions
 
Produce a roadmap document ready to paste into Notion. Include:
 
---
 
**Title**: [Project] Roadmap — [Scope/Version Range]
**Status**: Draft
**Created**: [today's date]
**Time horizon**: [e.g., Q2 2024 / next 3 months]
 
---
 
## Vision
One-paragraph description of where this project is heading. What does success look like at the end of this roadmap?
 
## Milestones
 
For each milestone:
 
### Milestone [N]: [Name]
- **Target**: [date or relative, e.g., "4 weeks from now"]
- **Goal**: What user value does this milestone deliver?
- **Key features**:
  - Feature A
  - Feature B
- **Exit criteria**: How do we know this milestone is done?
 
## Release Plan
 
| Version | Focus | Key Features | Target Date | Distribution |
|---------|-------|--------------|-------------|--------------|
| v0.x.x  |       |              |             | Internal / Beta / Public |
 
## Prioritization Rationale
Explain why features are ordered as they are. What's driving priority — user impact, technical dependency, risk reduction?
 
## Assumptions
List assumptions this roadmap depends on being true.
 
## Risks
What could cause timeline slippage? How will you mitigate?
 
---
 
Keep this roadmap realistic for a solo developer. Flag if any milestone looks too ambitious.

---

## Save to Notion

After generating the roadmap above, save it to the project's Notion space.

### Instructions

1. **Ask the user** for:
   - **Project name** — which project is this roadmap for (e.g., Task Tracker, Expense Tracker, Personal AI)
   - **Target version** — the version this roadmap targets (e.g., `1.2.0`)

2. **Find the project in Notion** — use `mcp__notion__notion-search` to locate the project's root page.

3. **Find the version page** — use `mcp__notion__notion-fetch` on the project page to list its children, then look for a child page matching the target version number (e.g., `1.2.0`).

4. **If the version page does not exist** — create it using `mcp__notion__notion-create-pages` as a child of the project page, with the title set to the version number (e.g., `1.2.0`).

5. **Create the roadmap page** — use `mcp__notion__notion-create-pages` as a child of the version page:
   - **Title**: `[Project] Roadmap — [Scope/Version Range]` (follow the Notion documentation standard)
   - **Icon**: 🗺️
   - **Content**: The full roadmap output above, formatted in Notion Markdown:
     - Include all sections: Vision, Milestones, Release Plan, Prioritization Rationale, Assumptions, Risks
     - Add metadata at the top: **Status**: Draft, **Created**: today, **Time horizon**: as specified

6. **Confirm to the user** — share the Notion page URL once created.
