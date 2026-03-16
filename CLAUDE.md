# Claude Global Context — Personal Workflow
 
## About Me
- **Role**: Senior Android Engineer
- **Primary stack**: Kotlin, Jetpack Compose, Android SDK
- **Secondary**: Backend integration, REST APIs, Firebase
- **Work mode**: Solo indie developer — I own every stage end-to-end
- **Context**: Personal projects only. Company work is managed separately.
 
## Active Personal Projects
| Project | Stage | Type |
|---|---|---|
| Task Tracker | Closed Beta | Android app |
| Expense Tracker | Closed Beta | Android app |
 
Each project has its own Notion space for all docs, tasks, and decisions.
 
## Communication Style
- **Language**: English for everything — code, docs, commits, PRDs, design notes, Notion
- Direct and concise — no fluff, no essay explanations
- Code blocks for all code, commands, file paths
- Multiple options? State trade-offs briefly, then recommend best one
- Never explain obvious things
 
## My Full Development Workflow
 
Every feature goes through these stages in order:
 
```
STAGE 1  IDEATION       → raw idea, problem statement, goals
STAGE 2  PRD            → requirements, user stories, acceptance criteria → Notion
STAGE 3  BREAKDOWN      → epics → features → milestones → release plan → Notion
STAGE 4  TASKS          → actionable tasks with estimates → Notion (tracked)
STAGE 5  DESIGN         → UI/UX concepts, component design, screen flows → Notion
STAGE 6  ARCHITECTURE   → approach decision, patterns, trade-offs, ADR → Notion
STAGE 7  IMPLEMENTATION → Android/Kotlin code, production quality
STAGE 8  CODE REVIEW    → self-review: logic, security, perf, edge cases
STAGE 9  TESTING        → unit tests, manual plan, UI tests, automation
STAGE 10 GIT & PR       → Conventional Commits, PR description, CI pass, merge
STAGE 11 RELEASE        → release notes, version bump, Play Store
STAGE 12 DOCUMENTATION  → document all learnings, decisions, changes → Notion
```
 
## Android Tech Stack (Default)
```
Language:      Kotlin
UI:            Jetpack Compose
Architecture:  MVVM or MVI (check project CLAUDE.md)
DI:            Hilt
Async:         Coroutines + Flow
Local DB:      Room
Network:       Retrofit + OkHttp
Navigation:    Compose Navigation
Image:         Coil
Testing:       JUnit4/5, MockK, Turbine, Compose Test Rule
Build:         Gradle KTS + version catalog (libs.versions.toml)
```
 
## Core Engineering Principles
1. **Production quality** — assume this fails in prod; handle it now
2. **Security first** — auth, data exposure, input validation, EncryptedSharedPreferences
3. **Offline-first** — local cache, sync strategy, graceful degradation
4. **Clean architecture** — domain layer owns business logic; zero Android deps in domain
5. **Test coverage** — unit test business logic and ViewModels; skip trivial boilerplate
6. **Incremental** — small focused PRs; never big bang rewrites
7. **Consistency** — follow existing project conventions before introducing new patterns
8. **Explicit over clever** — readable code beats smart code
 
## Git Conventions
```
Branch:
  feat/<short-description>
  fix/<short-description>
  chore/<short-description>
  docs/<short-description>
  test/<short-description>
  refactor/<short-description>
 
Commit (Conventional Commits):
  feat(scope): description
  fix(scope): description
  chore(scope): description
  docs(scope): description
  refactor(scope): description
  test(scope): description
  perf(scope): description
 
PR title  = same style as commit
PR body   = what / why / how / test plan
```
 
## Code Review Checklist
Before any code is considered ready:
- [ ] Logic correct + all edge cases handled
- [ ] No sensitive data in logs, error messages, or API responses
- [ ] Auth and permission checks in place
- [ ] No force `!!` unwrap without explicit justification
- [ ] Error handling: network, DB, and unknown/unexpected errors
- [ ] No coroutine scope leaks (tied to lifecycle correctly)
- [ ] No unnecessary recompositions or N+1 queries
- [ ] Unit tests cover critical business logic
- [ ] Naming is clear and self-documenting
 
## Notion Documentation Standard
Every doc saved to Notion:
- **Title format**: `[Project] [DocType] — [Feature/Topic]`
  e.g. `[TaskTracker] PRD — Recurring Tasks`
- **Date**: created + last updated
- **Status**: Draft | In Review | Approved | Archived
- **TL;DR**: 2–3 line summary at top
- **Decisions**: explicitly call out decisions made and WHY
- Tables and headers for structure
 
## Available Slash Commands
```
/idea       → Capture and evaluate a raw idea
/prd        → Write a Product Requirements Document
/breakdown  → Break a feature/epic into tasks + estimates
/roadmap    → Plan milestones and release timeline
/design     → UI/UX design concept: flows, components, screens
/arch       → Architecture Decision Record (ADR)
/impl       → Implementation plan for a feature/task
/review     → Self-code review against the full checklist
/test       → Test strategy and test plan
/pr         → Generate PR title + body
/release    → Generate release notes for a version
/doc        → Format notes/decisions into Notion-ready documentation
/standup    → Generate standup update from current context
```
 
## Default Behavior
When receiving any task I automatically:
1. Read the project `CLAUDE.md` if present — understand conventions first
2. Explore relevant files before writing any code
3. Think about edge cases, error states, and failure paths
4. Follow existing patterns; introduce new ones only when clearly better
5. Use English for all output
 
## When In Doubt
- Explore first, code second
- Ask 1 specific question, not multiple vague ones
- For multiple approaches: state trade-offs briefly, recommend best option
- If scope is unclear: make smallest correct change, flag what's left
