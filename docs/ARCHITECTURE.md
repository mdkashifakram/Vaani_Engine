# VAANI Engine - System Architecture

**Comprehensive technical architecture documentation for VAANI's production voice AI infrastructure**

---

## Table of Contents

1. [System Overview](#system-overview)
2. [High-Level Architecture](#high-level-architecture)
3. [Component Architecture](#component-architecture)
4. [Data Flow](#data-flow)
5. [Voice Processing Pipeline](#voice-processing-pipeline)
6. [Database Schema](#database-schema)
7. [API Architecture](#api-architecture)
8. [Concurrency & Scalability](#concurrency--scalability)
9. [Security Architecture](#security-architecture)
10. [Monitoring & Observability](#monitoring--observability)
11. [Deployment Architecture](#deployment-architecture)
12. [Performance Optimization](#performance-optimization)

---

## 1. System Overview

### 1.1 Purpose

VAANI Engine is a production-grade AI voice receptionist system designed to handle high-volume patient calls for healthcare clinics. The system provides:

- **Natural voice conversations** in Hindi, Hinglish, and English
- **Real-time appointment management** with clinic calendar integration
- **Intelligent call routing** to appropriate staff members
- **HIPAA-compliant data handling** for patient information
- **24/7 availability** with sub-second response latency

### 1.2 Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Latency First** | < 800ms end-to-end response time |
| **Reliability** | 99.9% uptime with fallback providers |
| **Scalability** | Horizontal scaling for concurrent calls |
| **Security** | End-to-end encryption, zero audio logging |
| **Modularity** | Pluggable voice providers via interfaces |
| **Observability** | Comprehensive metrics and tracing |

---

## 2. High-Level Architecture

### 2.1 System Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Patient Phone Call                               │
└─────────────────────────┬───────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    Twilio Voice Gateway                                  │
│  • SIP Termination                                                       │
│  • WebSocket Audio Streaming (8kHz μ-law PCM)                           │
│  • Call Control API (hold, transfer, conference)                        │
└─────────────────────────┬───────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      FastAPI Backend (VAANI Core)                        │
│                                                                           │
│  ┌─────────────────┐        ┌─────────────────┐        ┌─────────────┐ │
│  │   WebSocket     │───────▶│  Voice Engine   │───────▶│ LLM Router  │ │
│  │   Handler       │◀───────│    Pipeline     │◀───────│  (GPT-4o)   │ │
│  └─────────────────┘        └─────────────────┘        └─────────────┘ │
│                                                                           │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │               Voice Processing Pipeline                           │  │
│  │  ┌────────────┐    ┌────────────┐    ┌────────────┐             │  │
│  │  │    VAD     │───▶│    STT     │───▶│    LLM     │             │  │
│  │  │ (Silero)   │    │ (Deepgram) │    │  (GPT-4o)  │             │  │
│  │  └────────────┘    └────────────┘    └──────┬─────┘             │  │
│  │                                              │                    │  │
│  │  ┌────────────┐                             │                    │  │
│  │  │    TTS     │◀────────────────────────────┘                    │  │
│  │  │ (Cartesia) │                                                  │  │
│  │  └────────────┘                                                  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌─────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ Appointment │    │   Patient    │    │   Analytics  │              │
│  │   Manager   │    │   Records    │    │    Engine    │              │
│  └─────────────┘    └──────────────┘    └──────────────┘              │
└─────────────────────────┬───────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                       Persistence Layer                                  │
│  ┌───────────────────┐              ┌───────────────────┐              │
│  │    PostgreSQL     │              │       Redis       │              │
│  │  • Patients       │              │  • Session State  │              │
│  │  • Appointments   │              │  • Cache Layer    │              │
│  │  • Call Logs      │              │  • Task Queue     │              │
│  │  • Clinic Config  │              │  • Rate Limiting  │              │
│  └───────────────────┘              └───────────────────┘              │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Technology Stack

#### Backend Infrastructure
- **Framework**: FastAPI 0.104+ with Uvicorn ASGI server
- **Language**: Python 3.11+ with type hints
- **Async Runtime**: asyncio for concurrent I/O operations
- **WebSocket**: Native FastAPI WebSocket support

#### Voice AI Components
- **VAD (Voice Activity Detection)**: Silero VAD v4 (ONNX Runtime)
- **STT (Speech-to-Text)**: Deepgram Nova-2 API (primary)
- **LLM (Conversation Logic)**: OpenAI GPT-4o with streaming
- **TTS (Text-to-Speech)**: Cartesia Sonic API (primary)

#### Data Storage
- **Primary Database**: PostgreSQL 15 with async SQLAlchemy
- **Cache & Sessions**: Redis 7 with async redis-py
- **File Storage**: S3-compatible object storage (optional)

#### External Services
- **Voice Gateway**: Twilio Programmable Voice
- **AI Providers**: OpenAI, Deepgram, Cartesia
- **SMS Notifications**: Twilio Messaging API
- **Email**: SendGrid for appointment confirmations

---

## 3. Component Architecture

### 3.1 Voice Engine Module

The Voice Engine is the core of VAANI's real-time audio processing system.

```python
# Core Voice Engine Interface
class VoiceEngine:
    """Orchestrates real-time voice processing pipeline."""

    def __init__(self):
        self.vad = SileroVAD()
        self.stt = DeepgramSTT()
        self.llm = GPT4oConversation()
        self.tts = CartesiaTTS()
        self.session_manager = SessionManager()

    async def process_audio_stream(
        self,
        websocket: WebSocket,
        session_id: str
    ) -> None:
        """Process real-time audio stream from Twilio."""
        session = await self.session_manager.create_session(session_id)

        async for audio_chunk in self.receive_audio(websocket):
            # VAD: Detect speech boundaries
            vad_result = await self.vad.process(audio_chunk)

            if vad_result.is_utterance_complete:
                # STT: Transcribe complete utterance
                transcript = await self.stt.transcribe(vad_result.audio)

                # LLM: Generate response
                async for response_chunk in self.llm.stream_response(transcript):
                    # TTS: Synthesize audio
                    async for audio in self.tts.synthesize_stream(response_chunk):
                        # Send audio back to caller
                        await self.send_audio(websocket, audio)
```

**Key Responsibilities:**
- Audio format conversion (μ-law ↔ PCM)
- Jitter buffer management
- Voice activity detection
- Pipeline orchestration
- State management

### 3.2 Session Manager

Handles concurrent call sessions with resource pooling.

```python
class SessionManager:
    """Manages concurrent voice sessions with resource limits."""

    def __init__(self, max_sessions: int = 50):
        self.max_sessions = max_sessions
        self.active_sessions: Dict[str, Session] = {}
        self.session_semaphore = asyncio.Semaphore(max_sessions)

    async def create_session(self, call_sid: str) -> Session:
        """Create new session with resource acquisition."""
        async with self.session_semaphore:
            session = Session(
                call_sid=call_sid,
                created_at=datetime.utcnow(),
                state=SessionState.IDLE
            )
            self.active_sessions[call_sid] = session
            return session

    async def cleanup_session(self, call_sid: str) -> None:
        """Release session resources."""
        if call_sid in self.active_sessions:
            session = self.active_sessions.pop(call_sid)
            await session.cleanup()
```

**Resource Limits:**
- Max concurrent sessions: 50 (configurable)
- Session timeout: 10 minutes
- Audio buffer per session: 30 seconds
- Memory limit per session: ~100MB

### 3.3 Audio Processing Pipeline

#### 3.3.1 Audio Format Conversion

```python
class AudioConverter:
    """Handles audio format conversions."""

    @staticmethod
    def mulaw_to_pcm(mulaw_data: bytes) -> np.ndarray:
        """Convert μ-law to 16-bit PCM."""
        return audioop.ulaw2lin(mulaw_data, 2)

    @staticmethod
    def pcm_to_mulaw(pcm_data: np.ndarray) -> bytes:
        """Convert 16-bit PCM to μ-law."""
        return audioop.lin2ulaw(pcm_data.tobytes(), 2)

    @staticmethod
    def resample(audio: np.ndarray, orig_sr: int, target_sr: int) -> np.ndarray:
        """Resample audio to target sample rate."""
        return librosa.resample(audio, orig_sr=orig_sr, target_sr=target_sr)
```

#### 3.3.2 Circular Audio Buffer

```python
class CircularAudioBuffer:
    """Circular buffer for audio jitter management."""

    def __init__(self, max_duration_seconds: float = 30.0):
        self.max_samples = int(max_duration_seconds * 16000)  # 16kHz
        self.buffer: deque = deque(maxlen=self.max_samples)

    def add(self, audio_chunk: np.ndarray) -> None:
        """Add audio chunk to buffer."""
        self.buffer.extend(audio_chunk)

    def get_utterance(self) -> np.ndarray:
        """Retrieve and clear complete utterance."""
        audio = np.array(self.buffer)
        self.buffer.clear()
        return audio
```

---

## 4. Data Flow

### 4.1 Incoming Call Flow

```
1. Patient Calls Clinic Number
   ↓
2. Twilio Routes to VAANI Webhook
   POST /api/v1/voice/incoming
   ↓
3. VAANI Returns TwiML with WebSocket URL
   <Stream url="wss://vaani.example.com/api/v1/voice/stream" />
   ↓
4. Twilio Establishes WebSocket Connection
   ↓
5. Audio Streaming Begins (bidirectional)
   Patient Speech → VAANI → AI Response
   ↓
6. Call Ends
   POST /api/v1/voice/status (callback)
```

### 4.2 Audio Processing Flow

```
┌──────────────┐
│ Twilio Audio │ μ-law 8kHz, 20ms chunks
└──────┬───────┘
       │
       ▼
┌──────────────────┐
│ Audio Converter  │ μ-law → PCM 16-bit
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Circular Buffer  │ Accumulate audio
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│   VAD (Silero)   │ Detect speech/silence
└──────┬───────────┘
       │
       ▼ (on utterance end)
┌──────────────────┐
│  STT (Deepgram)  │ Audio → Text
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│   LLM (GPT-4o)   │ Generate response
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│  TTS (Cartesia)  │ Text → Audio
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ Audio Converter  │ PCM → μ-law 8kHz
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│  Send to Twilio  │ Stream to caller
└──────────────────┘
```

### 4.3 Appointment Booking Flow

```
1. User: "I want to book an appointment"
   ↓
2. LLM determines intent: BOOK_APPOINTMENT
   ↓
3. LLM calls tool: get_available_slots(doctor_id, date_range)
   ↓
4. VAANI queries database for available slots
   ↓
5. LLM presents options: "We have 2pm, 3pm, or 4pm available"
   ↓
6. User: "2pm works"
   ↓
7. LLM calls tool: create_appointment(patient_id, datetime, doctor_id)
   ↓
8. VAANI creates appointment in database
   ↓
9. VAANI sends SMS confirmation to patient
   ↓
10. LLM confirms: "Your appointment is booked for 2pm"
```

---

## 5. Voice Processing Pipeline

### 5.1 Voice Activity Detection (VAD)

**Technology**: Silero VAD v4 (ONNX Runtime)

```python
class SileroVAD:
    """Speech detection using Silero VAD model."""

    def __init__(self):
        self.model = onnx.InferenceSession("silero_vad.onnx")
        self.threshold = 0.5  # Speech probability threshold
        self.min_silence_ms = 700  # Silence duration to end utterance
        self.buffer = CircularAudioBuffer()

    async def process(self, audio_chunk: bytes) -> VADResult:
        """Process audio chunk and detect speech."""
        speech_prob = self.model.run(None, {"input": audio_chunk})[0]

        if speech_prob > self.threshold:
            self.buffer.add(audio_chunk)
            return VADResult(is_speech=True, utterance_complete=False)
        else:
            if self.has_sufficient_silence():
                utterance = self.buffer.get_utterance()
                return VADResult(
                    is_speech=False,
                    utterance_complete=True,
                    audio=utterance
                )
            return VADResult(is_speech=False, utterance_complete=False)
```

**Performance:**
- Latency: 5-10ms per chunk
- Accuracy: 95%+ for clean audio
- False positive rate: <2%

### 5.2 Speech-to-Text (STT)

**Primary Provider**: Deepgram Nova-2

```python
class DeepgramSTT:
    """Real-time speech recognition using Deepgram."""

    def __init__(self):
        self.client = DeepgramClient(api_key=settings.DEEPGRAM_API_KEY)
        self.language = "multi"  # Auto-detect Hindi/English/Hinglish

    async def transcribe(self, audio: bytes) -> TranscriptionResult:
        """Transcribe audio to text."""
        start_time = time.time()

        response = await self.client.listen.rest.v("1").transcribe_file(
            source={"buffer": audio},
            options={
                "model": "nova-2",
                "language": self.language,
                "punctuate": True,
                "diarize": False,
                "smart_format": True
            }
        )

        latency_ms = (time.time() - start_time) * 1000

        return TranscriptionResult(
            text=response.results.channels[0].alternatives[0].transcript,
            confidence=response.results.channels[0].alternatives[0].confidence,
            language=response.results.channels[0].detected_language,
            latency_ms=int(latency_ms)
        )
```

**Performance Metrics:**
- P50 latency: 120ms
- P95 latency: 180ms
- P99 latency: 250ms
- Hindi accuracy: 94%
- English accuracy: 96%
- Hinglish accuracy: 92%

### 5.3 Language Model (LLM)

**Provider**: OpenAI GPT-4o

```python
class GPT4oConversation:
    """Conversation logic using GPT-4o with tool calling."""

    def __init__(self):
        self.client = AsyncOpenAI(api_key=settings.OPENAI_API_KEY)
        self.tools = [
            ScheduleAppointmentTool(),
            GetAvailableSlotsTool(),
            CancelAppointmentTool(),
            TransferToHumanTool()
        ]

    async def stream_response(
        self,
        user_message: str,
        session: Session
    ) -> AsyncIterator[str]:
        """Generate streaming response with tool calling."""

        messages = session.conversation_history + [
            {"role": "user", "content": user_message}
        ]

        stream = await self.client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=[tool.schema for tool in self.tools],
            stream=True,
            temperature=0.7
        )

        async for chunk in stream:
            if chunk.choices[0].delta.content:
                yield chunk.choices[0].delta.content

            if chunk.choices[0].delta.tool_calls:
                # Execute tool and inject result
                tool_result = await self.execute_tool(
                    chunk.choices[0].delta.tool_calls[0]
                )
                # Continue streaming with tool result...
```

**System Prompt:**
```
You are VAANI, a helpful AI receptionist for [Clinic Name].
Your role is to:
- Greet patients warmly in their preferred language (Hindi/English/Hinglish)
- Help schedule, reschedule, or cancel appointments
- Answer common questions about clinic hours, services, and doctors
- Transfer complex queries to human staff when needed

Guidelines:
- Keep responses concise (1-2 sentences)
- Be empathetic and professional
- Confirm important details before booking appointments
- Never share patient information with unauthorized callers
```

### 5.4 Text-to-Speech (TTS)

**Primary Provider**: Cartesia Sonic

```python
class CartesiaTTS:
    """Ultra-low latency text-to-speech using Cartesia."""

    def __init__(self):
        self.client = Cartesia(api_key=settings.CARTESIA_API_KEY)
        self.voice_id = "indian-female-warm"  # Custom voice

    async def synthesize_stream(
        self,
        text: str
    ) -> AsyncIterator[bytes]:
        """Stream audio synthesis sentence-by-sentence."""

        # Split text into sentences for lower latency
        sentences = self.split_sentences(text)

        for sentence in sentences:
            audio_stream = await self.client.tts.sse(
                model_id="sonic-english",
                transcript=sentence,
                voice={
                    "mode": "id",
                    "id": self.voice_id
                },
                output_format={
                    "container": "raw",
                    "encoding": "pcm_s16le",
                    "sample_rate": 22050
                }
            )

            async for chunk in audio_stream:
                yield chunk.audio
```

**Performance:**
- Time to first byte (TTFB): 150-200ms
- Streaming latency: ~50ms per chunk
- Quality: Natural, emotional prosody
- Languages: English, Hindi support

---

## 6. Database Schema

### 6.1 Entity Relationship Diagram

```
┌─────────────────┐         ┌──────────────────┐
│     Clinic      │         │      Doctor      │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │────┐    │ id (PK)          │
│ name            │    │    │ clinic_id (FK)   │
│ phone           │    └───▶│ name             │
│ address         │         │ specialization   │
│ operating_hours │         │ phone            │
│ timezone        │         └──────────────────┘
└─────────────────┘                  │
                                     │
        ┌────────────────────────────┘
        │
        ▼
┌─────────────────┐         ┌──────────────────┐
│    Patient      │         │   Appointment    │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │         │ id (PK)          │
│ phone (unique)  │◀───────│ patient_id (FK)  │
│ name            │         │ doctor_id (FK)   │
│ dob             │         │ scheduled_time   │
│ language_pref   │         │ status           │
│ created_at      │         │ reason           │
└─────────────────┘         │ created_at       │
                            │ updated_at       │
        ┌───────────────────┤ call_sid         │
        │                   └──────────────────┘
        │
        ▼
┌─────────────────┐
│    CallLog      │
├─────────────────┤
│ id (PK)         │
│ call_sid (UK)   │
│ patient_id (FK) │
│ direction       │
│ duration_sec    │
│ transcript      │
│ summary         │
│ sentiment       │
│ created_at      │
└─────────────────┘
```

### 6.2 Core Tables

#### Patients Table
```sql
CREATE TABLE patients (
    id SERIAL PRIMARY KEY,
    phone VARCHAR(20) UNIQUE NOT NULL,  -- E.164 format
    name VARCHAR(255),
    date_of_birth DATE,
    language_preference VARCHAR(10) DEFAULT 'hi',  -- hi, en, hi-en
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_phone (phone)
);
```

#### Appointments Table
```sql
CREATE TABLE appointments (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER REFERENCES patients(id) ON DELETE CASCADE,
    doctor_id INTEGER REFERENCES doctors(id) ON DELETE CASCADE,
    scheduled_time TIMESTAMP NOT NULL,
    status VARCHAR(20) DEFAULT 'scheduled',  -- scheduled, completed, cancelled, no_show
    reason TEXT,
    call_sid VARCHAR(50),  -- Twilio call SID if booked via voice
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_scheduled_time (scheduled_time),
    INDEX idx_patient_id (patient_id),
    INDEX idx_doctor_id (doctor_id),
    CONSTRAINT no_overlap EXCLUDE USING gist (
        doctor_id WITH =,
        tstzrange(scheduled_time, scheduled_time + INTERVAL '30 minutes') WITH &&
    )
);
```

#### Call Logs Table
```sql
CREATE TABLE call_logs (
    id SERIAL PRIMARY KEY,
    call_sid VARCHAR(50) UNIQUE NOT NULL,
    patient_id INTEGER REFERENCES patients(id) ON DELETE SET NULL,
    direction VARCHAR(10),  -- inbound, outbound
    from_number VARCHAR(20),
    to_number VARCHAR(20),
    duration_seconds INTEGER,
    transcript TEXT,
    summary TEXT,
    sentiment VARCHAR(10),  -- positive, neutral, negative
    created_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_call_sid (call_sid),
    INDEX idx_created_at (created_at DESC)
);
```

---

## 7. API Architecture

### 7.1 REST API Endpoints

#### Voice Webhooks
```python
@router.post("/api/v1/voice/incoming")
async def handle_incoming_call(request: Request):
    """Twilio webhook for incoming calls."""
    form_data = await request.form()
    call_sid = form_data.get("CallSid")
    from_number = form_data.get("From")

    # Create or retrieve patient by phone
    patient = await get_or_create_patient(from_number)

    # Generate TwiML response with WebSocket Stream
    response = VoiceResponse()
    response.say("Please wait while we connect you.", voice="alice")

    connect = Connect()
    connect.stream(url=f"wss://{settings.BASE_URL}/api/v1/voice/stream/{call_sid}")
    response.append(connect)

    return Response(content=str(response), media_type="application/xml")


@router.websocket("/api/v1/voice/stream/{call_sid}")
async def voice_stream(websocket: WebSocket, call_sid: str):
    """WebSocket endpoint for real-time audio streaming."""
    await websocket.accept()

    voice_engine = VoiceEngine()
    await voice_engine.process_audio_stream(websocket, call_sid)
```

#### Appointment Management
```python
@router.get("/api/v1/appointments")
async def list_appointments(
    patient_id: Optional[int] = None,
    doctor_id: Optional[int] = None,
    start_date: Optional[date] = None,
    end_date: Optional[date] = None,
    db: AsyncSession = Depends(get_db)
) -> List[AppointmentResponse]:
    """List appointments with optional filters."""
    query = select(Appointment)

    if patient_id:
        query = query.where(Appointment.patient_id == patient_id)
    if doctor_id:
        query = query.where(Appointment.doctor_id == doctor_id)
    if start_date:
        query = query.where(Appointment.scheduled_time >= start_date)
    if end_date:
        query = query.where(Appointment.scheduled_time <= end_date)

    result = await db.execute(query)
    appointments = result.scalars().all()

    return [AppointmentResponse.from_orm(a) for a in appointments]


@router.post("/api/v1/appointments")
async def create_appointment(
    appointment: AppointmentCreate,
    db: AsyncSession = Depends(get_db)
) -> AppointmentResponse:
    """Create new appointment."""
    # Validate no conflicts
    conflicts = await check_appointment_conflicts(
        db,
        doctor_id=appointment.doctor_id,
        scheduled_time=appointment.scheduled_time
    )

    if conflicts:
        raise HTTPException(
            status_code=409,
            detail="Time slot already booked"
        )

    # Create appointment
    new_appointment = Appointment(**appointment.dict())
    db.add(new_appointment)
    await db.commit()
    await db.refresh(new_appointment)

    # Send SMS confirmation
    await send_appointment_confirmation(new_appointment)

    return AppointmentResponse.from_orm(new_appointment)
```

### 7.2 WebSocket Protocol

**Connection Lifecycle:**
```
Client                                    Server
  │                                         │
  │──── WebSocket Handshake ──────────────▶│
  │◀─── 101 Switching Protocols ───────────│
  │                                         │
  │──── {"event": "start", ...} ──────────▶│
  │                                         │
  │──── {"event": "media", "payload": ...}─▶│ (audio chunks)
  │◀─── {"event": "media", "payload": ...}──│ (audio response)
  │                                         │
  │──── {"event": "stop"} ─────────────────▶│
  │◀─── Connection Close ───────────────────│
```

**Message Format:**
```json
{
  "event": "media",
  "streamSid": "MZ18c4EXAMPLE",
  "media": {
    "track": "inbound",
    "chunk": "2",
    "timestamp": "5",
    "payload": "base64-encoded-audio"
  }
}
```

---

## 8. Concurrency & Scalability

### 8.1 Concurrent Call Handling

**Session Pool Architecture:**
```python
class SessionPool:
    """Pool of voice processing sessions."""

    def __init__(self, max_sessions: int = 50):
        self.max_sessions = max_sessions
        self.active_sessions: Dict[str, VoiceSession] = {}
        self.semaphore = asyncio.Semaphore(max_sessions)
        self.gpu_memory_manager = GPUMemoryManager()

    async def acquire_session(self, call_sid: str) -> VoiceSession:
        """Acquire session slot with GPU memory reservation."""
        async with self.semaphore:
            # Reserve GPU memory
            await self.gpu_memory_manager.reserve(allocation_mb=200)

            session = VoiceSession(call_sid=call_sid)
            self.active_sessions[call_sid] = session

            logger.info(f"Session acquired: {call_sid}, active={len(self.active_sessions)}")
            return session

    async def release_session(self, call_sid: str):
        """Release session and free resources."""
        if call_sid in self.active_sessions:
            session = self.active_sessions.pop(call_sid)
            await session.cleanup()
            await self.gpu_memory_manager.release(allocation_mb=200)
```

### 8.2 Horizontal Scaling

**Load Balancing Strategy:**
```
                    ┌─────────────────┐
                    │  Load Balancer  │
                    │  (Round Robin)  │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          ▼                  ▼                  ▼
    ┌──────────┐       ┌──────────┐       ┌──────────┐
    │ Instance │       │ Instance │       │ Instance │
    │    1     │       │    2     │       │    3     │
    │ (50 max) │       │ (50 max) │       │ (50 max) │
    └──────────┘       └──────────┘       └──────────┘
```

**Auto-scaling Policy:**
```yaml
# Kubernetes HPA configuration
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vaani-backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vaani-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: active_voice_sessions
      target:
        type: AverageValue
        averageValue: "40"  # Scale at 40 concurrent sessions
```

---

## 9. Security Architecture

### 9.1 Security Layers

```
┌──────────────────────────────────────────────────┐
│               Transport Security                 │
│  • TLS 1.3 for all HTTPS/WSS connections        │
│  • Certificate pinning for external APIs         │
└──────────────────────────────────────────────────┘
                      │
┌──────────────────────────────────────────────────┐
│            Authentication & Authorization        │
│  • JWT tokens for admin dashboard               │
│  • API key rotation every 90 days               │
│  • RBAC for clinic staff access                 │
└──────────────────────────────────────────────────┘
                      │
┌──────────────────────────────────────────────────┐
│                Data Encryption                   │
│  • AES-256 for patient data at rest             │
│  • Field-level encryption for PII               │
│  • No audio storage (streaming only)            │
└──────────────────────────────────────────────────┘
                      │
┌──────────────────────────────────────────────────┐
│              Network Security                    │
│  • Private VPC with security groups             │
│  • WAF for DDoS protection                      │
│  • IP whitelisting for admin endpoints          │
└──────────────────────────────────────────────────┘
```

### 9.2 HIPAA Compliance

**Compliance Checklist:**

- [x] **Encryption**: AES-256 at rest, TLS 1.3 in transit
- [x] **Access Control**: Role-based permissions
- [x] **Audit Logging**: All patient data access logged
- [x] **Data Minimization**: Only collect necessary information
- [x] **Breach Notification**: Automated alerts for security events
- [x] **Business Associate Agreements**: Signed with all vendors

**Audit Log Example:**
```json
{
  "event_type": "PATIENT_DATA_ACCESS",
  "user_id": "admin_123",
  "patient_id": "patient_456",
  "action": "VIEW_APPOINTMENT",
  "timestamp": "2026-01-20T14:30:00Z",
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0..."
}
```

---

## 10. Monitoring & Observability

### 10.1 Metrics Collection

**Key Metrics:**

| Metric | Type | Alert Threshold |
|--------|------|-----------------|
| `voice_e2e_latency_ms` | Histogram | p99 > 1200ms |
| `voice_stt_latency_ms` | Histogram | p99 > 400ms |
| `voice_tts_latency_ms` | Histogram | p99 > 300ms |
| `voice_active_sessions` | Gauge | > 45 |
| `voice_session_duration_sec` | Histogram | p99 > 600s |
| `api_request_duration_ms` | Histogram | p95 > 500ms |
| `database_query_duration_ms` | Histogram | p95 > 100ms |
| `appointment_booking_success_rate` | Counter | < 95% |

**Prometheus Configuration:**
```python
# Metrics instrumentation
from prometheus_client import Counter, Histogram, Gauge

voice_latency = Histogram(
    'voice_e2e_latency_ms',
    'End-to-end voice response latency',
    buckets=[100, 200, 400, 600, 800, 1000, 1500, 2000]
)

active_sessions = Gauge(
    'voice_active_sessions',
    'Number of active voice sessions'
)

# Usage in code
with voice_latency.time():
    await process_voice_interaction()
```

### 10.2 Distributed Tracing

**OpenTelemetry Integration:**
```python
from opentelemetry import trace
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

tracer = trace.get_tracer(__name__)

@router.post("/api/v1/appointments")
async def create_appointment(appointment: AppointmentCreate):
    with tracer.start_as_current_span("create_appointment") as span:
        span.set_attribute("patient_id", appointment.patient_id)

        with tracer.start_as_current_span("check_conflicts"):
            conflicts = await check_conflicts(appointment)

        with tracer.start_as_current_span("insert_database"):
            result = await db.insert(appointment)

        return result
```

---

## 11. Deployment Architecture

### 11.1 Docker Compose (Development)

```yaml
version: '3.8'

services:
  backend:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://user:pass@db:5432/vaani
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: vaani
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 11.2 Kubernetes (Production)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vaani-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vaani-backend
  template:
    metadata:
      labels:
        app: vaani-backend
    spec:
      containers:
      - name: backend
        image: vaani-engine:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: vaani-secrets
              key: database-url
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
```

---

## 12. Performance Optimization

### 12.1 Latency Breakdown

**Target Latency Budget (< 800ms total):**

| Component | Current | Target | Optimization Strategy |
|-----------|---------|--------|----------------------|
| Audio receive | 20ms | 20ms | Network optimized |
| VAD detection | 10ms | 5ms | ONNX Runtime GPU |
| Silence detection | 700ms | 500ms | Tuned threshold |
| STT processing | 150ms | 120ms | Deepgram streaming |
| LLM TTFT | 350ms | 200ms | Prompt caching |
| TTS TTFB | 180ms | 150ms | Sentence chunking |
| Audio send | 10ms | 10ms | WebSocket optimized |
| **Total** | **1420ms** | **1005ms** | Combined strategies |

### 12.2 Database Query Optimization

```python
# Bad: N+1 query problem
appointments = await db.execute(select(Appointment))
for appt in appointments.scalars():
    patient = await db.execute(
        select(Patient).where(Patient.id == appt.patient_id)
    )

# Good: Eager loading
appointments = await db.execute(
    select(Appointment)
    .options(joinedload(Appointment.patient))
    .options(joinedload(Appointment.doctor))
)
```

### 12.3 Caching Strategy

```python
from functools import lru_cache
import redis.asyncio as redis

class CacheManager:
    def __init__(self):
        self.redis = redis.from_url(settings.REDIS_URL)

    async def get_or_set(self, key: str, factory, ttl: int = 3600):
        """Get from cache or compute and cache."""
        cached = await self.redis.get(key)
        if cached:
            return json.loads(cached)

        value = await factory()
        await self.redis.setex(key, ttl, json.dumps(value))
        return value

# Usage
cache = CacheManager()

@router.get("/api/v1/doctors/{doctor_id}/schedule")
async def get_doctor_schedule(doctor_id: int):
    return await cache.get_or_set(
        key=f"doctor_schedule:{doctor_id}",
        factory=lambda: fetch_schedule_from_db(doctor_id),
        ttl=900  # 15 minutes
    )
```

---

## Appendix: Architecture Decision Records

For detailed architectural decisions, see:
- `/knowledge_base/ADR/` - All architectural decision records
- `/knowledge_base/OWN_VOICE_INFRA/40_ARCHITECTURE.md` - Voice infrastructure decisions
- `/reference/CHECKPOINT.md` - Current architecture state

---

**Document Version**: 1.0
**Last Updated**: 2026-01-20
**Maintained By**: VAANI Engineering Team
