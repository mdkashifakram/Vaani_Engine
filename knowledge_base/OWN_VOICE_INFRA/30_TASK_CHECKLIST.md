# Vaani Engine - Task Checklist

> **Purpose**: Granular task tracking for building production-grade voice infrastructure

---

## Legend
- `[ ]` Not started
- `[/]` In progress
- `[x]` Completed
- `[!]` Blocked

---

## Phase 1: Project Foundation

### 1.1 Environment Setup
- [ ] Create `vaani-engine/` project directory
- [ ] Initialize Python 3.11+ virtual environment
- [ ] Install PyTorch with GPU support (CUDA/MPS)
- [ ] Create `pyproject.toml` with project metadata
- [ ] Set up pre-commit hooks (black, ruff, mypy)

### 1.2 Core Infrastructure
- [ ] Create `app/config.py` with pydantic-settings
- [ ] Set up structured logging (structlog)
- [ ] Create metrics infrastructure (prometheus-client)
- [ ] Create `.env.example` with all configuration variables
- [ ] Write project README.md

### 1.3 Audio Utilities
- [ ] Implement `audio/converter.py` - μ-law ↔ PCM conversion
- [ ] Implement `audio/buffer.py` - Circular audio buffer
- [ ] Implement `audio/resampler.py` - Sample rate conversion
- [ ] Write unit tests for audio utilities
- [ ] Test with sample Twilio audio

---

## Phase 2: VAD Module

### 2.1 Silero VAD Integration
- [ ] Download Silero VAD ONNX model
- [ ] Create `vad/base.py` - Abstract VAD interface
- [ ] Implement `vad/silero_vad.py` - Silero wrapper
- [ ] Configure speech detection threshold (0.5-0.7)
- [ ] Implement end-of-utterance detection logic

### 2.2 VAD Optimization
- [ ] Add minimum speech duration filter (250ms)
- [ ] Implement silence padding (500-700ms)
- [ ] Add speech probability smoothing
- [ ] Test with noisy audio samples
- [ ] Benchmark VAD latency (<1ms target)

### 2.3 WebRTC VAD Fallback
- [ ] Implement `vad/webrtc_vad.py` - WebRTC fallback
- [ ] Add fallback mechanism
- [ ] Test fallback scenarios

---

## Phase 3: STT Module

### 3.1 faster-whisper Setup
- [ ] Create model download script
- [ ] Download Whisper large-v3 model (~3GB)
- [ ] Create `stt/base.py` - Abstract STT interface
- [ ] Implement `stt/whisper_stt.py` - faster-whisper wrapper
- [ ] Configure GPU/MPS acceleration

### 3.2 STT Features
- [ ] Implement streaming transcription
- [ ] Add partial result callback
- [ ] Implement language detection
- [ ] Add language routing (Hindi vs English)
- [ ] Test with Hindi audio samples

### 3.3 IndicWhisper Integration
- [ ] Download AI4Bharat IndicWhisper model
- [ ] Implement `stt/indic_stt.py` - IndicWhisper wrapper
- [ ] Test Hinglish transcription accuracy
- [ ] Compare accuracy: Whisper vs IndicWhisper

### 3.4 Deepgram Fallback
- [ ] Implement `stt/deepgram_stt.py` - API client
- [ ] Add timeout-based fallback (500ms)
- [ ] Implement retry with exponential backoff
- [ ] Test failover scenarios

### 3.5 STT Testing
- [ ] Write unit tests for Whisper STT
- [ ] Write accuracy tests (WER measurement)
- [ ] Benchmark STT latency (<300ms target)
- [ ] Test with 100+ sample utterances

---

## Phase 4: TTS Module

### 4.1 XTTS-v2 Setup
- [ ] Download XTTS-v2 model (~1.5GB)
- [ ] Create `tts/base.py` - Abstract TTS interface
- [ ] Implement `tts/xtts_tts.py` - XTTS-v2 wrapper
- [ ] Configure Hindi language support
- [ ] Set up default voice sample

### 4.2 TTS Streaming
- [ ] Implement streaming audio output
- [ ] Add sentence chunking logic
- [ ] Implement audio format conversion (22kHz → 8kHz)
- [ ] Test streaming playback

### 4.3 Voice Management
- [ ] Create voice sample storage
- [ ] Implement voice selection API
- [ ] Add voice sample validation
- [ ] Document voice sample requirements

### 4.4 Cartesia Fallback
- [ ] Implement `tts/cartesia_tts.py` - API client
- [ ] Add timeout-based fallback
- [ ] Match voice characteristics
- [ ] Test failover scenarios

### 4.5 TTS Testing
- [ ] Write unit tests for XTTS-v2
- [ ] Test Hindi TTS quality
- [ ] Benchmark TTS latency (TTFB <200ms)
- [ ] Collect subjective quality feedback

---

## Phase 5: Pipeline Integration

### 5.1 Voice Pipeline Orchestrator
- [ ] Create `core/voice_pipeline.py` - Main orchestrator
- [ ] Implement audio stream routing
- [ ] Add concurrent session management
- [ ] Implement state machine (listening → processing → speaking)

### 5.2 Interruption Handling
- [ ] Implement barge-in detection
- [ ] Add TTS cancellation on user speech
- [ ] Test conversation flow with interruptions

### 5.3 WebSocket Server
- [ ] Create `api/websocket.py` - WebSocket endpoint
- [ ] Define message protocol (JSON metadata + binary audio)
- [ ] Implement heartbeat/keepalive
- [ ] Add connection error handling
- [ ] Handle graceful disconnection

### 5.4 Session Management
- [ ] Create `core/session_manager.py`
- [ ] Implement session lifecycle (create → active → destroy)
- [ ] Add session timeout handling
- [ ] Implement resource cleanup

### 5.5 Pipeline Testing
- [ ] Write end-to-end tests
- [ ] Test concurrent sessions (5, 10, 20)
- [ ] Test long-running sessions (>5 min)
- [ ] Benchmark full pipeline latency

---

## Phase 6: VAANI Integration

### 6.1 Voice Engine Client
- [ ] Create VoiceEngineClient class in VAANI
- [ ] Implement WebSocket connection management
- [ ] Add audio stream bridging
- [ ] Handle reconnection logic

### 6.2 AudioBridge Modification
- [ ] Add voice engine routing
- [ ] Implement feature flag check
- [ ] Add latency logging

### 6.3 Configuration
- [ ] Add VOICE_ENGINE environment variable
- [ ] Add CUSTOM_VOICE_ENGINE_URL config
- [ ] Add VOICE_ENGINE_TIMEOUT config
- [ ] Update .env.example

### 6.4 Integration Testing
- [ ] Test with OpenAI (existing flow)
- [ ] Test with custom engine
- [ ] Test hybrid fallback
- [ ] Verify Twilio audio compatibility

---

## Phase 7: Optimization

### 7.1 Latency Optimization
- [ ] Profile each pipeline stage
- [ ] Optimize audio buffering
- [ ] Implement model warm-up
- [ ] Add ONNX optimization for VAD

### 7.2 Memory Optimization
- [ ] Profile GPU memory usage
- [ ] Implement model unloading (idle sessions)
- [ ] Optimize concurrent session limit

### 7.3 Benchmarking
- [ ] Create `scripts/benchmark_latency.py`
- [ ] Measure p50, p90, p99 latencies
- [ ] Compare with OpenAI baseline
- [ ] Document performance results

---

## Phase 8: Production Deployment

### 8.1 Containerization
- [ ] Create `docker/Dockerfile` (CPU)
- [ ] Create `docker/Dockerfile.gpu` (GPU)
- [ ] Create `docker-compose.yml`
- [ ] Test container builds
- [ ] Configure model volume mounting

### 8.2 Cloud Setup
- [ ] Choose GPU provider (RunPod/Lambda Labs)
- [ ] Create deployment script
- [ ] Configure SSL/TLS
- [ ] Set up domain/DNS

### 8.3 Monitoring
- [ ] Set up Prometheus metrics endpoint
- [ ] Create Grafana dashboard
- [ ] Configure alerting rules
- [ ] Set up log aggregation

### 8.4 Documentation
- [ ] Write deployment guide
- [ ] Document API endpoints
- [ ] Create troubleshooting guide
- [ ] Document scaling procedures

---

## Phase 9: Future Enhancements

### 9.1 Voice Cloning
- [ ] Collect 30+ min voice recordings
- [ ] Set up Google Colab training notebook
- [ ] Fine-tune XTTS-v2 on custom voice
- [ ] Test cloned voice quality

### 9.2 CosyVoice2 Migration
- [ ] Evaluate CosyVoice2 Hindi support
- [ ] Implement CosyVoice2 wrapper
- [ ] Compare latency with XTTS-v2
- [ ] Plan migration if beneficial

### 9.3 Regional Language Expansion
- [ ] Identify priority languages (Tamil, Telugu, Bengali)
- [ ] Evaluate model support for each
- [ ] Plan data collection for training
- [ ] Set timeline for expansion

### 9.4 Advanced Features
- [ ] Implement emotion detection
- [ ] Add speaking style control
- [ ] Implement speaker diarization
- [ ] Add real-time translation

---

## Dependencies & Prerequisites

### Software Requirements
| Software | Version | Purpose |
|----------|---------|---------|
| Python | 3.11+ | Runtime |
| PyTorch | 2.1+ | ML framework |
| CUDA | 12.0+ | GPU acceleration |
| Docker | 24+ | Containerization |
| ffmpeg | 6+ | Audio processing |

### Model Downloads
| Model | Size | Source |
|-------|------|--------|
| Whisper large-v3 | ~3GB | HuggingFace |
| XTTS-v2 | ~1.5GB | Coqui AI |
| Silero VAD | ~2MB | GitHub |
| IndicWhisper | ~3GB | AI4Bharat |

### API Keys Required
| Provider | Purpose | Required When |
|----------|---------|---------------|
| Deepgram | STT fallback | Always |
| Cartesia | TTS fallback | Always |
| OpenAI | GPT-4 Turbo | LLM |

---

## Progress Tracking

| Phase | Status | Start Date | End Date | Notes |
|-------|--------|------------|----------|-------|
| 1. Foundation | Not Started | — | — | — |
| 2. VAD | Not Started | — | — | — |
| 3. STT | Not Started | — | — | — |
| 4. TTS | Not Started | — | — | — |
| 5. Pipeline | Not Started | — | — | — |
| 6. Integration | Not Started | — | — | — |
| 7. Optimization | Not Started | — | — | — |
| 8. Production | Not Started | — | — | — |
