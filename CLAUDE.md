# Claude Global Context — Personal Workflow
 
## About Me
- **Role**: Senior Mobile Engineer (Android + iOS) + Python AI Engineer
- **Android stack**: Kotlin, Jetpack Compose, Android SDK
- **iOS stack**: Swift, SwiftUI, iOS SDK
- **Python/AI stack**: Python, FastAPI, Ollama, async/await
- **Work mode**: Solo indie developer — I own every stage end-to-end
- **Context**: Personal projects only. Company work is managed separately.
 
## Active Personal Projects
| Project | Stage | Platforms |
|---|---|---|
| Task Tracker | Closed Beta | Android + iOS |
| Expense Tracker | Closed Beta | Android + iOS |
| Personal AI | Active Dev | Python (Ollama-powered) |
 
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
STAGE 7  IMPLEMENTATION → Android/iOS code, production quality
STAGE 8  CODE REVIEW    → self-review: logic, security, perf, edge cases
STAGE 9  TESTING        → unit tests, manual plan, UI tests, automation
STAGE 10 GIT & PR       → Conventional Commits, PR description, CI pass, merge
STAGE 11 RELEASE        → release notes, version bump, Play Store / App Store
STAGE 12 DOCUMENTATION  → document all learnings, decisions, changes → Notion
```
 
## Android Tech Stack
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
 
## iOS Tech Stack
```
Language:      Swift
UI:            SwiftUI
Architecture:  MVVM with @Observable (check project CLAUDE.md)
DI:            Manual DI / DIContainer
Async:         async/await + Combine (where needed)
Local DB:      SwiftData (or Core Data for legacy)
Network:       URLSession + Codable (Alamofire if complex)
Navigation:    NavigationStack
Image:         Kingfisher / AsyncImage
Testing:       XCTest, XCUITest, protocol-based mocks
Build:         Xcode + SPM (Swift Package Manager)
Secrets:       Keychain (never UserDefaults for sensitive data)
```
 
## Python / AI Tech Stack
```
Language:      Python 3.11+
API:           FastAPI + Uvicorn
LLM:           Ollama (local models: llama3, mistral, codellama)
Validation:    Pydantic v2
Async:         asyncio + async/await
Local DB:      SQLAlchemy (async) + SQLite / PostgreSQL
Package mgr:   uv (preferred) or pip + pyproject.toml
Testing:       pytest + pytest-asyncio + httpx
Linting:       ruff + mypy
Config:        pydantic-settings + .env
```
 
## Core Engineering Principles
1. **Production quality** — assume this fails in prod; handle it now
2. **Security first** — auth, data exposure, input validation; Keychain (iOS) / EncryptedSharedPreferences (Android)
3. **Offline-first** — local cache, sync strategy, graceful degradation
4. **Clean architecture** — domain layer owns business logic; zero platform deps in domain
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
- [ ] No force unwrap (`!!` Kotlin / `!` Swift) without explicit justification
- [ ] Error handling: network, DB, and unknown/unexpected errors
- [ ] No scope/memory leaks (lifecycle-aware, no retain cycles)
- [ ] No unnecessary recompositions (Compose) / re-renders (SwiftUI) or N+1 queries
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
 
### Shared (platform-agnostic)
```
/idea       → Capture and evaluate a raw idea
/prd        → Write a Product Requirements Document
/breakdown  → Break a feature/epic into tasks + estimates
/roadmap    → Plan milestones and release timeline
/design     → UI/UX design concept: flows, components, screens
/arch       → Architecture Decision Record (ADR)
/pr         → Generate PR title + body
/release    → Generate release notes for a version
/doc        → Format notes/decisions into Notion-ready documentation
/standup    → Generate standup update from current context
```
 
### Android
```
/impl       → Android implementation plan (Kotlin/Compose/MVVM)
/review     → Android self-code review checklist
/test       → Android test plan (JUnit + MockK + Turbine + Compose)
/debug      → Android debug session (ADB, Logcat, Coroutines)
/perf       → Android performance audit (Compose, Room, memory)
/deps       → Gradle dependency management (libs.versions.toml)
```
 
### iOS
```
/ios-impl   → iOS implementation plan (Swift/SwiftUI/MVVM)
/ios-review → iOS self-code review checklist
/ios-test   → iOS test plan (XCTest + SwiftData + XCUITest)
/ios-debug  → iOS debug session (Xcode, Instruments, Swift Concurrency)
/ios-perf   → iOS performance audit (SwiftUI, SwiftData, memory)
/ios-deps   → SPM dependency management
```
 
### Python / AI
```
/py-impl    → Python/AI implementation plan (FastAPI + Ollama)
/py-review  → Python/AI code review (async, security, prompt injection)
/py-test    → Python test plan (pytest + asyncio + mocked Ollama)
/py-debug   → Python/AI debug session (async, Ollama, streaming)
/py-perf    → Python/AI performance audit (TTFT, async, memory)
/py-deps    → Python dependency management (uv / pyproject.toml)
```
 
## Default Behavior
When receiving any task I automatically:
1. Read the project `CLAUDE.md` if present — understand conventions first
2. Identify platform context (Android or iOS) from file types or explicit mention
3. Explore relevant files before writing any code
4. Think about edge cases, error states, and failure paths
5. Follow existing patterns; introduce new ones only when clearly better
6. Use English for all output
 
## When In Doubt
- Explore first, code second
- Ask 1 specific question, not multiple vague ones
- For multiple approaches: state trade-offs briefly, recommend best option
- If scope is unclear: make smallest correct change, flag what's left
- If platform is unclear: ask which platform (Android, iOS, or Python/AI) before implementing
