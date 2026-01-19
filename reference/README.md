# Reference Folder

This folder contains context documents that help maintain consistency across AI-assisted development sessions.

## Files

| File | Purpose | When to Update |
|------|---------|----------------|
| `PERSONA.md` | AI assistant behavior preferences | When you want different AI behavior |
| `GUARDRAILS.md` | Hard rules that should never be violated | Rarely - only for fundamental changes |
| `CHECKPOINT.md` | Current project state and decisions | Every sprint / major milestone |
| `SESSION_LOG.md` | Accountability log of dev sessions | End of every significant session |
| `ANALYSIS_FRAMEWORK.md` | Critical analysis checklist | Before major changes (reference only) |

## How It Works

1. **CLAUDE.md** (in project root) is read automatically at session start
2. CLAUDE.md references these files for detailed context
3. Use `/context-refresh` workflow to re-read everything during a session

## Quick Reference

- Need to change AI behavior? → Edit `PERSONA.md`
- Need to add a hard rule? → Edit `GUARDRAILS.md`  
- Starting new sprint? → Update `CHECKPOINT.md`
- Feeling context drift? → Run `/context-refresh`
