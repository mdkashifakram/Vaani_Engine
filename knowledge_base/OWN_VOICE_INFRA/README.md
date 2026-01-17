# Vaani Engine

> **Open-source voice AI infrastructure** — Production-grade STT, TTS, and VAD for Indian languages

```
GitHub: github.com/mdkashifakram/vaani-engine
Package: pip install vaani-engine
```

---

## 📋 Document Index

| # | Document | Description | Status |
|---|----------|-------------|--------|
| 01 | [Voice Migration Roadmap](./01_VOICE_MIGRATION_ROADMAP.md) | Original migration options from V2 | Reference |
| 02 | [Voice Migration Summary](./02_VOICE_MIGRATION_SUMMARY.md) | Quick overview of migration | Reference |
| 03 | [Custom Voice Pipeline Reference](./03_CUSTOM_VOICE_PIPELINE_REFERENCE.md) | V2 pipeline guide | Reference |
| 04 | [Implementation Plan Reference](./04_IMPLEMENTATION_PLAN_REFERENCE.md) | V2 implementation details | Reference |
| 05 | [Task Checklist Reference](./05_TASK_CHECKLIST_REFERENCE.md) | V2 task list | Reference |
| **10** | [**Research Analysis**](./10_RESEARCH_ANALYSIS.md) | Deep dive on open-source models | ⭐ Start Here |
| **20** | [**Implementation Plan**](./20_IMPLEMENTATION_PLAN.md) | V3 technical implementation guide | ⭐ Core Doc |
| **30** | [**Task Checklist**](./30_TASK_CHECKLIST.md) | Granular task tracking (100+ items) | ⭐ Core Doc |
| **40** | [**Architecture**](./40_ARCHITECTURE.md) | System design & diagrams | ⭐ Core Doc |
| **50** | [**Latency Optimization**](./50_LATENCY_OPTIMIZATION.md) | Performance tuning guide | ⭐ Core Doc |

---

## 🎯 Project Objectives

1. **Replicate OpenAI Realtime API** with open-source components
2. **Support Hindi, English, Hinglish** (expand to regional languages later)
3. **Achieve <800ms end-to-end latency** (stretch goal: <600ms)
4. **Production-grade reliability** with hybrid cloud fallback
5. **Voice cloning capability** (future phase)

---

## 🏗️ Technology Stack

### Primary Stack (Self-Hosted)

| Component | Technology | Why Chosen |
|-----------|------------|------------|
| **VAD** | [Silero VAD](https://github.com/snakers4/silero-vad) | Best accuracy, <1ms latency, free |
| **STT** | [faster-whisper](https://github.com/SYSTRAN/faster-whisper) + [AI4Bharat](https://ai4bharat.iitm.ac.in/) | Best Hindi support, GPU optimized |
| **TTS** | [XTTS-v2](https://github.com/coqui-ai/TTS) | Native Hindi, voice cloning, streaming |
| **LLM** | GPT-4 Turbo | Current VAANI integration, quality |

### Fallback Stack (Cloud APIs)

| Component | Fallback | When Used |
|-----------|----------|-----------|
| **STT** | [Deepgram](https://deepgram.com/) | Timeout (>500ms) or error |
| **TTS** | [Cartesia](https://cartesia.ai/) | Timeout (>300ms) or error |

---

## 📊 Key Metrics

| Metric | OpenAI Realtime | Our Target | Stretch Goal |
|--------|-----------------|------------|--------------|
| E2E Latency | ~400ms | <800ms | <600ms |
| STT Latency | ~100ms | <300ms | <200ms |
| TTS TTFB | ~150ms | <200ms | <150ms |
| Hindi Accuracy | Limited | >90% WER | >95% WER |
| Voice Quality | Good | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 📅 Timeline

| Week | Phase | Deliverables |
|------|-------|--------------|
| 1 | Foundation | Project setup, audio utilities |
| 1-2 | VAD | Silero VAD integration |
| 2-3 | STT | faster-whisper + IndicWhisper |
| 3-4 | TTS | XTTS-v2, streaming output |
| 4-5 | Pipeline | Full orchestration, WebSocket |
| 5-6 | Integration | VAANI connection, feature flags |
| 6-7 | Testing | Benchmarks, optimization |
| 7-8 | Production | Deployment, monitoring |

**Total: 8 weeks**

---

## 💰 Budget Estimate

| Phase | Monthly Cost |
|-------|--------------|
| **Development** | ~$70/month (GPU + API testing) |
| **Production** | ~$130-180/month (600 calls/month) |

---

## 🚀 Getting Started

1. **Read the Research Analysis** → Understand model options
2. **Review the Implementation Plan** → Understand the approach
3. **Use the Task Checklist** → Track progress
4. **Reference Architecture** → System design
5. **Apply Latency Optimization** → Performance tuning

---

## 📁 Related Projects

- **VAANI V2** - Current production system with OpenAI Realtime
- **vaani-engine** - The new voice engine (to be created)

---

## 📚 References

### Open-Source Models
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) - GPU-optimized Whisper
- [AI4Bharat](https://ai4bharat.iitm.ac.in/) - Indian language models
- [XTTS-v2](https://github.com/coqui-ai/TTS) - Multilingual TTS
- [Silero VAD](https://github.com/snakers4/silero-vad) - Voice activity detection

### Frameworks
- [Pipecat](https://github.com/pipecat-ai/pipecat) - Voice AI orchestration
- [LiveKit Agents](https://github.com/livekit/agents) - WebRTC + Voice AI

### Commercial Fallbacks
- [Deepgram](https://deepgram.com/) - STT API
- [Cartesia](https://cartesia.ai/) - TTS API
