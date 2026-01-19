# Development Guardrails

> Hard rules that should NEVER be violated during development.

---

## 🔴 Critical Rules (Never Break)

### Security
- [ ] **Never commit secrets** - API keys, passwords, tokens go in `.env` only
- [ ] **Never log PII** - No patient names, phone numbers, health info in logs
- [ ] **Never disable auth** - Even for "quick testing"
- [ ] **Never trust user input** - Validate and sanitize everything

### Data Integrity
- [ ] **Never modify production data directly** - Use migrations
- [ ] **Never delete without soft-delete first** - Especially patient records
- [ ] **Never bypass transaction boundaries** - atomic operations only
- [ ] **Never orphan records** - Cascade deletes properly configured

### Voice Pipeline
- [ ] **Never block the audio stream** - All processing must be async
- [ ] **Never exceed 800ms latency target** - Flag if unavoidable
- [ ] **Never drop audio without logging** - Debug requires traces
- [ ] **Never skip VAD** - Silence detection is critical for UX

---

## 🟡 Strong Guidelines (Deviate with Justification)

### Architecture
- Prefer composition over inheritance
- One responsibility per module
- Dependency injection for testability
- Feature flags for risky changes

### Code Quality
- Type hints on all public functions
- Docstrings on complex logic
- Integration tests for critical paths
- Error messages should be actionable

### Performance
- Profile before optimizing
- Cache at the edge (Redis) not in app
- Batch database operations
- Stream large payloads

---

## 🟢 Preferences (Flexible)

- Prefer `asyncio` over threading
- Prefer `pydantic` for validation
- Prefer explicit over magic
- Prefer boring technology

---

## Escalation Matrix

| Issue Type | Action Required |
|------------|-----------------|
| Security vulnerability | STOP. Notify immediately. |
| Data loss risk | STOP. Get explicit approval. |
| Breaking change | Create implementation plan first |
| Performance regression | Quantify impact before proceeding |
| Dependency addition | Justify necessity |

---

## Pre-Commit Checklist

Before any PR/commit:
- [ ] No secrets in code
- [ ] No PII in logs
- [ ] Tests pass
- [ ] Migrations reversible
- [ ] Latency impact documented (if applicable)
