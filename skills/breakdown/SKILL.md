# /breakdown — Feature & Task Breakdown
 
Break down the following feature/epic into actionable tasks: **$ARGUMENTS**
 
## Instructions
 
Produce a task breakdown ready to paste into Notion. Format as follows:
 
---
 
**Title**: [Project] Breakdown — [Feature/Epic Name]
**Status**: Draft
**Created**: [today's date]
 
---
 
## Summary
1–2 sentences describing what this breakdown covers.
 
## Epics → Features → Tasks
 
Group tasks by epic or functional area. For each task:
 
```
[ ] Task name
    Type: feat | fix | chore | test | docs
    Estimate: XS (< 1h) | S (1–3h) | M (3–6h) | L (1–2d) | XL (> 2d)
    Notes: any important context, dependencies, or risks
```
 
## Dependency Map
Which tasks must be completed before others can start? List blockers clearly.
 
## Estimates Summary
| Size | Count | Rough Total |
|------|-------|-------------|
| XS   |       |             |
| S    |       |             |
| M    |       |             |
| L    |       |             |
| XL   |       |             |
| **Total** |  |             |
 
## Risks & Unknowns
Flag anything that could expand scope or cause delays. Be honest.
 
## Out of Scope (v1)
Anything explicitly deferred to a future iteration.
 
---
 
After the breakdown, identify the 1–2 highest-risk tasks and explain why they're risky.

---

## Save to Notion

After generating the breakdown above, save it to the project's Notion space.

### Instructions

1. **Ask the user** for:
   - **Project name** — which project is this breakdown for (e.g., Task Tracker, Expense Tracker, Personal AI)
   - **Target version** — the version this breakdown targets (e.g., `1.2.0`)

2. **Find the project in Notion** — use `mcp__notion__notion-search` to locate the project's root page.

3. **Find the version page** — use `mcp__notion__notion-fetch` on the project page to list its children, then look for a child page matching the target version number (e.g., `1.2.0`).

4. **If the version page does not exist** — create it using `mcp__notion__notion-create-pages` as a child of the project page, with the title set to the version number (e.g., `1.2.0`).

5. **Create the breakdown page** — use `mcp__notion__notion-create-pages` as a child of the version page:
   - **Title**: `[Project] Breakdown — [Feature/Epic Name]` (follow the Notion documentation standard)
   - **Icon**: 🔨
   - **Content**: The full breakdown output above, formatted in Notion Markdown:
     - Include all sections: Summary, Epics → Features → Tasks, Dependency Map, Estimates Summary, Risks & Unknowns, Out of Scope
     - Add metadata at the top: **Status**: Draft, **Created**: today

6. **Confirm to the user** — share the Notion page URL once created.

---

## Create Task Board

After saving the breakdown document, create a Notion task board database to track implementation progress.

### Instructions

1. **Create the task board database** — use `mcp__notion__notion-create-database` as a child of the version page (the same version page used above):
   - **Title**: `[Project] Tasks — [Feature/Epic Name]`
   - **Schema**:
     ```
     CREATE TABLE (
       "Task" TITLE,
       "Status" SELECT('To Do':default, 'In Progress':blue, 'In Review':yellow, 'Done':green, 'Blocked':red),
       "Type" SELECT('feat':blue, 'fix':red, 'chore':default, 'test':green, 'docs':purple),
       "Estimate" SELECT('XS':default, 'S':green, 'M':yellow, 'L':orange, 'XL':red),
       "Epic" SELECT(<populate with epic/functional area names from the breakdown>),
       "Blocked By" RICH_TEXT,
       "Notes" RICH_TEXT
     )
     ```

2. **Populate tasks** — use `mcp__notion__notion-create-pages` to add all tasks from the breakdown as rows in the database:
   - Set **Task** to the task name
   - Set **Status** to `To Do`
   - Set **Type** from the breakdown (feat / fix / chore / test / docs)
   - Set **Estimate** from the breakdown (XS / S / M / L / XL)
   - Set **Epic** to the epic/functional area the task belongs to
   - Set **Blocked By** from the dependency map (if any)
   - Set **Notes** from the task notes (if any)

3. **Create a board view** — use `mcp__notion__notion-create-view` on the database:
   - **Name**: `Board`
   - **Type**: `board`
   - **Configure**: `GROUP BY "Status"`

4. **Confirm to the user** — share the task board URL. This board is now the source of truth for tracking implementation progress via `/impl`.
