# 🤖 Repo 16 — `assistant-service`

> **Status:** Planned  
> **Flag:** —  
> **Difficulty:** ⭐⭐⭐⭐  
> **Build Priority:** 17 (Phase 4 — Intelligence)

---

## Purpose

The **multimodal AI assistant** — the conversational interface of GlucoVision that patients interact with via voice or camera. It consolidates voice Q&A, vision-based food Q&A, and cooking guidance into one assistant because these are three UI modes of one multimodal experience, not three separate products. A patient can say "What's in this food?" while pointing the camera — that requires voice + vision working together seamlessly.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `voice/` | Whisper ASR: speech → text → LLaMA-3 intent → TTS response |
| `vision_qa/` | Patient points camera at food → LLaMA-3 Vision answers "What is this? Is it safe for me?" |
| `multimodal_fusion/` | Combines voice + camera input into unified context for LLaMA-3 |
| `cooking_context/` | Receives cooking state from `11` → narrates cooking guidance to patient |
| `session_manager/` | Maintains conversation history per patient session via WebSocket |
| `safety_filter/` | Ensures LLM never gives medication dosage advice (guardrails) |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Voice audio stream | Mobile microphone (03) | PCM / WebM audio |
| Camera image (vision Q&A) | Mobile camera (03) | JPEG |
| Recommendation context | `15` recommendation-engine | JSON (current meal plan) |
| Cooking state | `11` cooking-ai | JSON `{ state, step }` |
| Patient health context | `06` user-service | JSON (diabetes type, restrictions) |
| Text message | Web dashboard chat (04) | JSON string |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Voice response (TTS) | Mobile app (03) | MP3 / PCM audio stream |
| Text response | Mobile app (03), Web dashboard (04) | JSON string |
| Cooking guidance narration | Mobile app (03) via WebSocket | JSON + audio |
| Vision Q&A answer | Mobile app (03) | JSON `{ answer, food_safety_score }` |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `15` recommendation-engine | Dietary context for Q&A |
| `11` cooking-ai | Cooking state for guidance narration |
| `06` user-service | Patient health context for personalized responses |
| `08` api-gateway | WebSocket proxying |
| Whisper | OpenAI Whisper ASR (speech → text) |
| LLaMA-3 Vision | Multimodal LLM (text + image) |
| TTS (Coqui / ElevenLabs) | Text → speech for voice responses |

---

## Communication

| Protocol | Used For |
|---|---|
| WebSocket | Real-time audio stream + response (voice assistant session) |
| REST / HTTPS | Vision Q&A (image upload + text response) |
| Internal HTTP | Fetch context from `15`, `11`, `06` |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) + WebSocket |
| ASR | OpenAI Whisper (faster-whisper for CPU optimization) |
| LLM | LLaMA-3 Vision (multimodal) via Ollama or Together AI |
| TTS | Coqui TTS (local) / ElevenLabs API |
| LLM Framework | LangChain (conversation memory) |
| Audio Processing | librosa, soundfile, ffmpeg |
| Containerization | Docker |
| GPU | Optional for faster Whisper + LLaMA-3 inference |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **Whisper ASR** | faster-whisper (CTranslate2 optimized) — multilingual (English + Sinhala approximation) |
| **LLaMA-3 Vision** | Receives image + text prompt → "This is hoppers, a Sri Lankan rice flour pancake. For T2D: moderate portion (2 plain hoppers). Avoid egg hoppers if controlling fat." |
| **TTS** | Coqui XTTS-v2 supports Sinhala-accented English output |
| **Conversation memory** | LangChain ConversationBufferWindowMemory — keep last 10 turns |
| **Safety guardrails** | System prompt: "Never advise on insulin dosage, medication changes, or emergency procedures — refer to clinician." |
| **Multimodal fusion** | Combine Whisper transcription + camera frame → single LLaMA-3 prompt with image |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| In-memory (per session) | Conversation history | LangChain memory object |
| PostgreSQL | Session logs (for clinician review) | JSON |
| MinIO | Temporary audio file cache | MP3 / WAV |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| WebSocket | Sticky session per patient conversation |
| GPU | LLaMA-3 Vision benefits from GPU for < 2s response |
| Whisper | CPU sufficient for faster-whisper; GPU for real-time |
| Audio streaming | WebSocket binary frame streaming for low-latency voice |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Audio privacy | Voice recordings not stored — transcribed then discarded |
| Image privacy | Vision Q&A images not stored permanently |
| Guardrails | LLM safety filter on all outputs — reject medication dosage answers |
| Auth | JWT validation via gateway |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Voice round-trip | Audio → Whisper → LLM → TTS → audio: target < 3s |
| Vision Q&A | Image → LLaMA-3 Vision → text: target < 2s |
| WebSocket | Reconnect with conversation history restore |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| ASR Tests | Whisper accuracy on Sri Lankan English accent test set |
| Vision Tests | LLaMA-3 Vision identifies Sri Lankan food correctly |
| Safety Tests | Medication advice prompt → assert rejected |
| Latency Tests | End-to-end voice response < 3s |
| Session Tests | Conversation history maintained across 10-turn session |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Response latency, session duration, ASR confidence, vision confidence |
| Logging | Session start/end, query type (voice/vision), response time |
| Alerts | LLM guardrail triggered → log for safety review |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🔴 Advanced | Multimodal LLM (vision + language), real-time voice pipeline (ASR → LLM → TTS), WebSocket streaming, multilingual considerations |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Whisper ASR integration for voice interfaces |
| LLaMA-3 Vision multimodal Q&A |
| Real-time TTS voice response generation |
| LangChain conversation memory management |
| LLM safety guardrails for medical applications |
| WebSocket-based voice assistant session |
| Multilingual (English / Sinhala) health AI |
