---
description: Refresh context by re-reading all reference documents to prevent drift
---

# Context Refresh Workflow

Use this workflow when you feel the AI has lost context or is drifting from project guidelines.

## When to Use

- Starting a new session after a break
- Before making a major architectural decision
- When the AI seems to have "forgotten" prior decisions
- When you're switching between different parts of the project

## Steps

### 1. Read Core Guardrails
Read the main project context file:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/CLAUDE.md
```

### 2. Read Current Checkpoint
Get the latest project state and decisions:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/reference/CHECKPOINT.md
```

### 3. Read Persona (if needed)
If AI behavior seems off:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/reference/PERSONA.md
```

### 4. Read Guardrails (if needed)
If AI is violating rules:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/reference/GUARDRAILS.md
```

### 5. Read Session Log (recent entries)
For accountability and continuity:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/reference/SESSION_LOG.md
```

### 6. Read Module Registry (when working on modules)
To understand codebase structure:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/knowledge_base/MODULE_REGISTRY.md
```

### 7. Read Analysis Framework (before major changes)
For critical analysis checklist:
```
view_file /Users/mdkashifakram/Desktop/PROJECTS/AI/VAANI_Engine/reference/ANALYSIS_FRAMEWORK.md
```

### 8. Acknowledge Context
After reading, summarize:
- Current phase/sprint
- Key decisions made
- What should NOT be changed
- What's the immediate next step

## Quick Version

For rapid context refresh, just read CHECKPOINT.md - it has the essential state.

## Updating Checkpoint

If you notice the checkpoint is stale, update it with:
- Current phase/sprint name
- What's completed
- What's in progress
- Recent decisions
- Any blockers
