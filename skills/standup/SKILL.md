# /standup — Standup Update
 
Generate a standup / progress update from current context.
 
## Instructions
 
Look at recent git commits, any open files, and the current task context, then produce a concise personal standup. This is for personal tracking — saves to Notion dev log.
 
```
git log --since="24 hours ago" --oneline
git status
git stash list
```
 
---
 
## Standup Format
 
**Date**: [today]
**Project**: [Project name]
 
### ✅ Done (since last standup)
What was actually completed? Reference commits or tasks.
- ...
 
### 🔨 In Progress
What is currently being worked on?
- ...
  - Blockers: [none / specific blocker]
  - ETA: [rough estimate]
 
### 📋 Next
What's planned for the next session?
- ...
 
### 🚨 Blockers / Risks
Anything blocking progress or raising concern?
- ...
 
### 💡 Notes / Learnings
Any decision made, insight gained, or thing worth remembering from today's session?
- ...
 
---
 
Keep this brief and honest. It's for your own record and to maintain momentum between sessions.
