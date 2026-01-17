# Vaani Engine - System Architecture

> **Purpose**: Technical architecture documentation for production-grade voice AI infrastructure

---

## 1. System Overview

### 1.1 High-Level Architecture

```mermaid
flowchart TB
    subgraph "External Services"
        Twilio[Twilio Media Streams]
        OpenAI[OpenAI GPT-4 Turbo]
    end
    
    subgraph "Voice Engine (vaani-voice-engine)"
        direction TB
        WS[WebSocket Server<br/>FastAPI + Uvicorn]
        
        subgraph "Audio Processing"
            Converter[Audio Converter<br/>μ-law ↔ PCM]
            Buffer[Circular Buffer<br/>Jitter Management]
        end
        
        subgraph "VAD Module"
            SileroVAD[Silero VAD<br/>Primary]
            WebRTCVAD[WebRTC VAD<br/>Fallback]
        end
        
        subgraph "STT Module"
            WhisperSTT[faster-whisper<br/>Primary]
            IndicSTT[IndicWhisper<br/>Hindi Optimized]
            DeepgramSTT[Deepgram API<br/>Fallback]
        end
        
        subgraph "TTS Module"
            XTTS[XTTS-v2<br/>Primary]
            CosyVoice[CosyVoice2<br/>Future]
            Cartesia[Cartesia API<br/>Fallback]
        end
        
        Pipeline[Voice Pipeline<br/>Orchestrator]
        SessionMgr[Session Manager<br/>Concurrency Control]
    end
    
    subgraph "VAANI Backend"
        Bridge[AudioBridge]
        Agent[VAANI Agent<br/>+ Tools]
    end
    
    Twilio <-->|WebSocket<br/>mulaw/8kHz| WS
    WS --> Converter
    Converter --> Buffer
    Buffer --> SileroVAD
    SileroVAD --> WhisperSTT
    WhisperSTT -->|Transcript| Pipeline
    Pipeline <-->|Text| Agent
    Agent <-->|API| OpenAI
    Pipeline --> XTTS
    XTTS --> Converter
    Converter --> WS
```

### 1.2 Component Responsibilities

| Component | Responsibility | Technology |
|-----------|----------------|------------|
| **WebSocket Server** | Connection management, audio streaming | FastAPI, websockets |
| **Audio Converter** | μ-law ↔ PCM, sample rate conversion | NumPy, SciPy |
| **Circular Buffer** | Jitter management, audio accumulation | Python deque |
| **VAD** | Speech detection, utterance boundaries | Silero VAD, ONNX |
| **STT** | Audio → Text transcription | faster-whisper, PyTorch |
| **TTS** | Text → Audio synthesis | XTTS-v2, PyTorch |
| **Pipeline** | Flow orchestration, state management | asyncio |
| **Session Manager** | Concurrent call handling | asyncio semaphore |

---

## 2. Data Flow

### 2.1 Audio Input Flow (User Speech → Transcript)

```mermaid
sequenceDiagram
    participant T as Twilio
    participant WS as WebSocket
    participant C as Converter
    participant B as Buffer
    participant V as VAD
    participant S as STT
    participant P as Pipeline
    
    T->>WS: μ-law audio (8kHz, 20ms chunks)
    WS->>C: Convert to PCM
    C->>B: Buffer PCM (16kHz)
    
    loop Every 30ms
        B->>V: Audio chunk (30ms)
        V-->>P: Speech probability
        
        alt Speech detected
            P->>B: Continue buffering
        else Silence detected (>700ms)
            B->>S: Complete utterance
            S-->>P: Transcript
        end
    end
```

### 2.2 Audio Output Flow (Response → User)

```mermaid
sequenceDiagram
    participant P as Pipeline
    participant L as LLM (GPT-4)
    participant TTS as TTS Engine
    participant C as Converter
    participant WS as WebSocket
    participant T as Twilio
    
    P->>L: User transcript
    L-->>P: Response text (streaming)
    
    loop Each sentence
        P->>TTS: Sentence text
        TTS-->>P: Audio chunk (streaming)
        P->>C: PCM → μ-law
        C->>WS: μ-law chunk
        WS->>T: Audio to caller
    end
```

---

## 3. Module Architecture

### 3.1 VAD Module

```mermaid
classDiagram
    class VADInterface {
        <<abstract>>
        +process_chunk(audio: bytes) VADResult
        +is_speaking() bool
        +get_utterance() Optional~bytes~
        +reset() void
    }
    
    class SileroVAD {
        -model: OnnxModel
        -sample_rate: int
        -threshold: float
        -buffer: deque
        +process_chunk(audio: bytes) VADResult
    }
    
    class WebRTCVAD {
        -vad: webrtcvad.Vad
        -aggressiveness: int
        +process_chunk(audio: bytes) VADResult
    }
    
    class VADResult {
        +speech_probability: float
        +is_speech: bool
        +is_utterance_end: bool
        +utterance: Optional~bytes~
    }
    
    VADInterface <|-- SileroVAD
    VADInterface <|-- WebRTCVAD
    SileroVAD ..> VADResult
    WebRTCVAD ..> VADResult
```

### 3.2 STT Module

```mermaid
classDiagram
    class STTInterface {
        <<abstract>>
        +transcribe(audio: bytes) TranscriptionResult
        +transcribe_stream(stream: AsyncIterator) AsyncIterator
    }
    
    class WhisperSTT {
        -model: WhisperModel
        -compute_type: str
        -language: str
        +transcribe(audio: bytes) TranscriptionResult
    }
    
    class IndicWhisperSTT {
        -model: IndicASR
        +transcribe(audio: bytes) TranscriptionResult
    }
    
    class DeepgramSTT {
        -client: DeepgramClient
        -api_key: str
        +transcribe(audio: bytes) TranscriptionResult
    }
    
    class TranscriptionResult {
        +text: str
        +language: str
        +confidence: float
        +latency_ms: int
    }
    
    STTInterface <|-- WhisperSTT
    STTInterface <|-- IndicWhisperSTT
    STTInterface <|-- DeepgramSTT
```

### 3.3 TTS Module

```mermaid
classDiagram
    class TTSInterface {
        <<abstract>>
        +synthesize(text: str) bytes
        +synthesize_stream(text: str) AsyncIterator~bytes~
        +set_voice(voice_id: str) void
    }
    
    class XTTS_TTS {
        -model: TTS
        -speaker_wav: str
        -language: str
        +synthesize(text: str) bytes
        +synthesize_stream(text: str) AsyncIterator~bytes~
    }
    
    class CosyVoiceTTS {
        -model: CosyVoice
        +synthesize_stream(text: str) AsyncIterator~bytes~
    }
    
    class CartesiaTTS {
        -client: CartesiaClient
        -voice_id: str
        +synthesize_stream(text: str) AsyncIterator~bytes~
    }
    
    TTSInterface <|-- XTTS_TTS
    TTSInterface <|-- CosyVoiceTTS
    TTSInterface <|-- CartesiaTTS
```

---

## 4. Voice Pipeline State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle: Connection established
    
    Idle --> Listening: Audio received
    Listening --> Listening: Speech detected
    Listening --> Processing: Silence detected (utterance end)
    
    Processing --> Thinking: STT complete
    Thinking --> Speaking: LLM response started
    
    Speaking --> Speaking: Sending audio chunks
    Speaking --> Listening: Response complete
    Speaking --> Listening: Barge-in detected
    
    Listening --> Idle: Long silence
    Idle --> [*]: Disconnect
```

### State Descriptions

| State | Description | Timeout |
|-------|-------------|---------|
| **Idle** | Connected, waiting for audio | 30s → disconnect |
| **Listening** | Receiving user speech | 10s → idle |
| **Processing** | Running STT on utterance | 5s → timeout error |
| **Thinking** | Waiting for LLM response | 10s → timeout error |
| **Speaking** | Streaming TTS audio | Until complete |

---

## 5. Hybrid Fallback Architecture

### 5.1 Fallback Decision Flow

```mermaid
flowchart TD
    A[Audio Input] --> B{Primary STT}
    B -->|Success < 500ms| C[Use Result]
    B -->|Timeout / Error| D{Fallback STT}
    D -->|Success| C
    D -->|Failure| E[Error Response]
    
    C --> F[LLM Processing]
    F --> G{Primary TTS}
    G -->|Success < 300ms| H[Stream Audio]
    G -->|Timeout / Error| I{Fallback TTS}
    I -->|Success| H
    I -->|Failure| E
```

### 5.2 Fallback Configuration

```python
class FallbackConfig(BaseSettings):
    # STT Fallback
    stt_primary: str = "whisper"          # whisper | indic
    stt_fallback: str = "deepgram"        # deepgram | assemblyai
    stt_timeout_ms: int = 500
    stt_retry_count: int = 1
    
    # TTS Fallback
    tts_primary: str = "xtts"             # xtts | cosyvoice
    tts_fallback: str = "cartesia"        # cartesia | elevenlabs
    tts_timeout_ms: int = 300
    tts_retry_count: int = 1
    
    # Circuit Breaker
    failure_threshold: int = 5
    recovery_timeout_s: int = 30
```

---

## 6. Concurrency Model

### 6.1 Session Architecture

```mermaid
flowchart TB
    subgraph "Session Pool"
        S1[Session 1<br/>Active Call]
        S2[Session 2<br/>Active Call]
        S3[Session 3<br/>Idle]
        SN[Session N<br/>...]
    end
    
    subgraph "Shared Resources"
        GPU[GPU Memory Pool]
        STT_Model[Whisper Model<br/>Shared]
        TTS_Model[XTTS Model<br/>Shared]
        VAD_Model[Silero VAD<br/>Shared]
    end
    
    S1 --> GPU
    S2 --> GPU
    STT_Model --> GPU
    TTS_Model --> GPU
```

### 6.2 Resource Limits

| Resource | Limit | Reasoning |
|----------|-------|-----------|
| Max concurrent sessions | 10-20 | GPU memory bound |
| Audio buffer per session | 30 seconds | Memory efficiency |
| Session timeout | 10 minutes | Call duration limit |
| GPU memory ceiling | 80% of available | Leave headroom |

---

## 7. Audio Format Specifications

### 7.1 Format Standards

| Stage | Format | Sample Rate | Bit Depth | Channels |
|-------|--------|-------------|-----------|----------|
| Twilio Input | μ-law | 8 kHz | 8-bit | Mono |
| Internal Processing | PCM | 16 kHz | 16-bit | Mono |
| Whisper Input | PCM | 16 kHz | 16-bit | Mono |
| XTTS Input | PCM | 22.05 kHz | 16-bit | Mono |
| XTTS Output | PCM | 22.05 kHz | 16-bit | Mono |
| Twilio Output | μ-law | 8 kHz | 8-bit | Mono |

### 7.2 Conversion Pipeline

```mermaid
flowchart LR
    A[Twilio μ-law<br/>8kHz] --> B[Decode μ-law]
    B --> C[Resample 16kHz]
    C --> D[STT Processing]
    D --> E[LLM]
    E --> F[TTS Processing<br/>22kHz]
    F --> G[Resample 8kHz]
    G --> H[Encode μ-law]
    H --> I[Twilio Output]
```

---

## 8. Latency Optimization Points

### 8.1 Optimization Opportunities

```mermaid
flowchart LR
    subgraph "Optimizations"
        O1[VAD: ONNX Runtime]
        O2[STT: GPU Batching]
        O3[STT: Partial Results]
        O4[TTS: Sentence Chunking]
        O5[TTS: Streaming Output]
        O6[Network: Warm Connections]
        O7[Memory: Model Pre-loading]
    end
```

### 8.2 Latency Breakdown Target

| Stage | Current Est. | Target | Technique |
|-------|--------------|--------|-----------|
| Audio receive | 20ms | 20ms | — |
| μ-law decode | 1ms | 1ms | — |
| Resample | 2ms | 2ms | — |
| VAD check | 10ms | 5ms | ONNX |
| Silence detection | 700ms | 500ms | Tuning |
| STT processing | 400ms | 250ms | GPU + partial |
| LLM TTFT | 300ms | 200ms | Prompt optimization |
| TTS TTFB | 300ms | 150ms | Streaming |
| Resample + encode | 5ms | 5ms | — |
| **Total** | **~1.7s** | **~1.1s** | — |

---

## 9. Deployment Architecture

### 9.1 Local Development

```mermaid
flowchart TB
    subgraph "Developer Machine (Mac M4)"
        VAANI[VAANI Backend<br/>:8000]
        Engine[Voice Engine<br/>:8001]
        ngrok[ngrok tunnel]
    end
    
    Twilio[Twilio] <--> ngrok
    ngrok <--> VAANI
    VAANI <--> Engine
    Engine <--> OpenAI[GPT-4 API]
```

### 9.2 Production Deployment

```mermaid
flowchart TB
    subgraph "Cloud Infrastructure"
        LB[Load Balancer]
        
        subgraph "GPU Instance"
            Engine1[Voice Engine 1]
            Engine2[Voice Engine 2]
        end
        
        subgraph "API Instance"
            VAANI1[VAANI Backend 1]
            VAANI2[VAANI Backend 2]
        end
        
        Redis[(Redis<br/>Session State)]
    end
    
    Twilio[Twilio] <--> LB
    LB <--> VAANI1
    LB <--> VAANI2
    VAANI1 <--> Engine1
    VAANI2 <--> Engine2
    Engine1 <--> Redis
    Engine2 <--> Redis
```

---

## 10. Monitoring Architecture

### 10.1 Metrics Collection

```mermaid
flowchart LR
    subgraph "Voice Engine"
        M1[VAD Latency]
        M2[STT Latency]
        M3[TTS Latency]
        M4[E2E Latency]
        M5[Active Sessions]
        M6[GPU Memory]
    end
    
    subgraph "Monitoring Stack"
        Prometheus[Prometheus]
        Grafana[Grafana]
        AlertManager[AlertManager]
    end
    
    M1 --> Prometheus
    M2 --> Prometheus
    M3 --> Prometheus
    M4 --> Prometheus
    M5 --> Prometheus
    M6 --> Prometheus
    
    Prometheus --> Grafana
    Prometheus --> AlertManager
```

### 10.2 Key Metrics

| Metric | Type | Alert Threshold |
|--------|------|-----------------|
| `voice_e2e_latency_ms` | Histogram | p99 > 1500ms |
| `voice_stt_latency_ms` | Histogram | p99 > 600ms |
| `voice_tts_latency_ms` | Histogram | p99 > 400ms |
| `voice_active_sessions` | Gauge | > 15 |
| `voice_gpu_memory_percent` | Gauge | > 85% |
| `voice_fallback_rate` | Counter | > 10% |

---

## 11. Security Considerations

### 11.1 Security Layers

| Layer | Protection |
|-------|------------|
| **Transport** | TLS 1.3 for all WebSocket |
| **Authentication** | Session tokens, API keys |
| **Audio Data** | No logging of audio content |
| **API Keys** | Environment variables, secrets manager |
| **Network** | Private VPC, firewall rules |

### 11.2 Data Handling

- Audio data is **not persisted** (streaming only)
- Transcripts can be **optionally logged** (configurable)
- LLM requests contain **no PII** in system prompts
- Fallback APIs use **encrypted connections**
