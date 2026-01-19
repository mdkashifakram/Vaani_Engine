# Critical Analysis Framework

> Checklist to apply before any significant change to VAANI Engine.
> 
> **Purpose**: Ensure thoughtful, reversible, well-documented decisions.

---

## When to Use This Framework

Apply this analysis when:
- Adding a new module or major feature
- Changing database schema
- Modifying voice pipeline
- Adding external dependencies
- Making breaking API changes
- Architectural refactoring

---

## Pre-Change Checklist

### 1. Impact Assessment

- [ ] **Blast radius**: What components could this break?
- [ ] **User impact**: Does this affect live calls or patient data?
- [ ] **Migration**: Is data migration required?
- [ ] **Downtime**: Any service interruption expected?

### 2. Scalability Check

- [ ] **Volume**: Will this work at 1M+ lines of code?
- [ ] **Performance**: Any O(n²) or worse complexity introduced?
- [ ] **Concurrency**: Thread-safe? Async-safe?
- [ ] **Memory**: Bounded memory usage?

### 3. Dependency Analysis

- [ ] **Upstream**: What does this depend on?
- [ ] **Downstream**: What depends on this?
- [ ] **External**: New third-party dependencies?
- [ ] **Versioning**: Breaking changes to existing contracts?

### 4. Reversibility

- [ ] **Rollback**: Can we undo this change?
- [ ] **Feature flag**: Should this be behind a flag?
- [ ] **Migration down**: Does the migration have a reverse?
- [ ] **Data backup**: Is backup needed before proceeding?

### 5. Test Coverage

- [ ] **Unit tests**: Core logic tested?
- [ ] **Integration tests**: Component interactions tested?
- [ ] **Edge cases**: Error paths and boundaries covered?
- [ ] **Manual verification**: What needs human validation?

### 6. Documentation

- [ ] **ADR**: Does this need an Architecture Decision Record?
- [ ] **Module Registry**: New module to register?
- [ ] **CHECKPOINT.md**: Update sprint state?
- [ ] **knowledge_base/**: Update architectural docs?

---

## Risk Matrix

| Probability \ Impact | Low | Medium | High |
|---------------------|-----|--------|------|
| **High** | Monitor | Plan mitigation | STOP - Get approval |
| **Medium** | Accept | Document | Plan mitigation |
| **Low** | Accept | Accept | Monitor |

---

## Quick Assessment Template

```markdown
## Change: [Brief description]

### Impact
- Blast radius: [Components affected]
- User impact: [None / Minimal / Significant]

### Risks
- [ ] Risk 1: [Description] - Mitigation: [Plan]

### Reversibility
- Rollback plan: [How to undo]
- Time to rollback: [Estimate]

### Decision
- [ ] Proceed
- [ ] Proceed with caution (flag enabled)
- [ ] Needs more analysis
- [ ] Blocked - requires approval
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It's Bad | Better Approach |
|--------------|--------------|-----------------|
| "Just ship it" | Accumulates technical debt | Apply this checklist |
| "We'll fix it later" | Later never comes | Fix now or document as TODO with owner |
| "It works on my machine" | Ignores environment differences | Test in staging |
| "No one uses that code" | Assumptions are dangerous | Check dependents first |
| "Quick hotfix" | Often causes more bugs | Root cause analysis |
