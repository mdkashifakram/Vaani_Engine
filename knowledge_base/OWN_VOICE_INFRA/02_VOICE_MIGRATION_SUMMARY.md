# Voice Provider Migration Roadmap

> **Purpose**: Replace OpenAI Realtime API with Whisper + GPT-4 + ElevenLabs for better voice quality

## Key Benefits
- Natural, emotional voice (not robotic)
- Hindi/Indian accent support
- Custom voice cloning option

## Architecture Change
Current: `Twilio → OpenAI Realtime (all-in-one) → Twilio`
New: `Twilio → Whisper STT → GPT-4 → ElevenLabs TTS → Twilio`

## Trade-offs
| Aspect | Current | New |
|--------|---------|-----|
| Latency | ~400ms | ~800-1200ms |
| Voice Quality | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Monthly Cost | ~$60 | ~$115 |

## Files to Change
- REPLACE `openai_realtime.py` → `voice_pipeline.py`
- MODIFY `voice.py` to use new pipeline
- ADD `elevenlabs_client.py`, `whisper_client.py`, `vad.py`

## Timeline
- Week 1-2: ElevenLabs + VAD setup
- Week 3-4: Full integration + testing

## See Full Roadmap
[voice_migration_roadmap.md](file:///Users/mdkashifakram/.gemini/antigravity/brain/037bd92c-e54a-42be-a58b-e3b3b13b9a34/voice_migration_roadmap.md)
