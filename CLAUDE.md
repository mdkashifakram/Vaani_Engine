# VAANI Project Context

> **This file is automatically read at the start of every Claude Code session.**

---

## Project Identity

**VAANI** (Voice Assistant for AI-powered Natural Interactions) is an AI voice receptionist system for healthcare clinics. The core mission is to handle appointment bookings, patient inquiries, and call routing with a natural, empathetic voice.

---

## 🚨 Immutable Guardrails

1. **Never break the Twilio ↔ AI voice bridge** - This is production-critical
2. **Never commit secrets** - All API keys go in `.env` (which is `.gitignore`'d)
3. **Always check `/reference/CHECKPOINT.md`** before making architectural decisions
4. **Always update `/knowledge_base`** when changing core architecture
5. **Phased implementation** - Follow the roadmap; don't skip phases

---

## Current Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Twilio     │────▶│   FastAPI    │────▶│   Voice AI   │
│ (Phone/SMS)  │◀────│  Backend     │◀────│   Pipeline   │
└──────────────┘     └──────────────┘     └──────────────┘
                            │
                     ┌──────▼──────┐
                     │  PostgreSQL │
                     │   + Redis   │
                     └─────────────┘
```

---

## Decision-Making Framework

Before ANY significant decision, verify:

| Check | How |
|-------|-----|
| Does this align with current phase? | Read `/reference/CHECKPOINT.md` |
| Was this already decided? | Check `/knowledge_base/OWN_VOICE_INFRA/` |
| Is this breaking change? | Flag in implementation plan |
| Does this affect voice latency? | Document expected impact |

---

## Current Sprint Context

> Read `/reference/CHECKPOINT.md` for the latest state.

**Major Initiative**: Voice Provider Migration (OpenAI Realtime → Custom STT/TTS Pipeline)

**Key Files**:
- `/knowledge_base/OWN_VOICE_INFRA/01_VOICE_MIGRATION_ROADMAP.md` - Full roadmap
- `/knowledge_base/OWN_VOICE_INFRA/30_TASK_CHECKLIST.md` - Task tracking

---

## Code Standards

- **Python**: 3.11+, FastAPI, async/await everywhere
- **Database**: PostgreSQL with SQLAlchemy async
- **Testing**: pytest with `pytest-asyncio`
- **Voice**: Target < 800ms end-to-end latency

---

## When in Doubt

1. Ask clarifying questions before implementing
2. Create an implementation plan and get approval
3. Check existing docs in `/knowledge_base/`
4. Refresh context with `/context-refresh` workflow
