# Custom Voice Pipeline - Task Checklist

## Overview
Build a custom STT ↔ TTS voice pipeline as a separate project, train on custom voice, then integrate with VAANI.

---

## Phase 1: Project Setup & Research
- [ ] Create new project structure (`vaani-voice-engine/`)
- [ ] Set up Python environment with PyTorch + MPS/CUDA support
- [ ] Research and finalize model choices (STT + TTS)

## Phase 2: STT Component
- [ ] Implement `faster-whisper` for Speech-to-Text
- [ ] Add Voice Activity Detection (VAD)
- [ ] Create audio buffering logic
- [ ] Build STT WebSocket endpoint

## Phase 3: TTS Component
- [ ] Set up XTTS-v2 or Fish Speech
- [ ] Implement streaming audio output
- [ ] Build TTS WebSocket endpoint

## Phase 4: Voice Training
- [ ] Record/collect voice samples
- [ ] Set up Google Colab training notebook
- [ ] Train custom voice model
- [ ] Export and integrate trained model

## Phase 5: Pipeline Integration
- [ ] Build unified WebSocket server (STT ↔ LLM ↔ TTS)
- [ ] Add audio format conversion (Twilio μ-law ↔ PCM)
- [ ] Integrate with VAANI's AudioBridge

## Phase 6: Testing & Optimization
- [ ] Latency benchmarking
- [ ] Quality testing with real calls
- [ ] Feature flag for India vs US/UK routing
