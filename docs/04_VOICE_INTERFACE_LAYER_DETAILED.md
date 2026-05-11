# 04 - Voice Interface Layer (Detailed Architecture)

## 1. Overview
The Voice Interface Layer is the primary interaction channel for users, handling the complete voice conversation flow without requiring UI interaction. This layer implements a speech-first paradigm with natural language understanding and contextually appropriate responses.

**Key Principle**: User speaks → System understands intent & context → System responds naturally

---

## 2. Core Components Architecture

### 2.1 Speech-to-Text (STT) Module
**Responsibility**: Convert user voice input to text with high accuracy in multilingual context

#### Technology Options:

| Component | Technology | Pros | Cons | Use Case |
|-----------|-----------|------|------|----------|
| **Cloud-Based** | Google Speech-to-Text API | Highest accuracy (~95%) | Internet required, costs | Primary system |
| **Cloud-Based** | Azure Speech Services | Good accuracy, cheap | Needs API key | Fallback option |
| **Local/Lightweight** | Whisper (OpenAI) | 99% accuracy, multilingual | Requires GPU | High-accuracy offline |
| **Local/Lightweight** | Vosk | Lightweight, offline | Lower accuracy (~80%) | Offline fallback |
| **Mobile-Optimized** | Native iOS/Android APIs | Built-in, efficient | Limited multilingual | Mobile app |

#### Recommended Stack:
```
Primary: Google Speech-to-Text (English, Sinhala, Tamil support)
Secondary: Whisper (offline mode, better Sinhala/Tamil)
Tertiary: Vosk (extreme lightweight)
```

#### Implementation Details:

**Configuration for Multilingual Support**:
```python
STT_CONFIG = {
    'english': {
        'language_code': 'en-US',
        'audio_encoding': 'LINEAR16',
        'sample_rate_hertz': 16000
    },
    'sinhala': {
        'language_code': 'si-LK',
        'audio_encoding': 'LINEAR16',
        'sample_rate_hertz': 16000
    },
    'tamil': {
        'language_code': 'ta-IN',
        'audio_encoding': 'LINEAR16',
        'sample_rate_hertz': 16000
    }
}
```

**Accuracy Expectations**:
- English: 95-98%
- Sinhala: 85-92% (limited training data)
- Tamil: 80-90% (mixed speech challenges)

#### Error Handling:
```
If STT_CONFIDENCE < 0.7:
  → Ask user to repeat
  → Suggest: "Could you please repeat that?"

If SINHALA/TAMIL detected but accuracy < 0.65:
  → Fallback to asking written input via UI
  
If MULTILINGUAL MIX detected:
  → Process as code-switching (handle Sinhala+English mix)
```

---

### 2.2 Natural Language Understanding (NLU) Module
**Responsibility**: Extract user intent, entities, and context from transcribed text

#### Intent Recognition System

**Core Intents** (Based on Diabetes Management):
```
1. LOG_FOOD
   Input: "I had rice and chicken curry"
   Extracted: meal_type=rice|chicken_curry, time=now

2. LOG_ACTIVITY
   Input: "I walked for 30 minutes"
   Extracted: activity=walking, duration=30min

3. CHECK_PREDICTION
   Input: "What will my glucose be?"
   Extracted: prediction_request=true, timing=near_future

4. GET_RECOMMENDATION
   Input: "Is this food healthy?"
   Extracted: food_query=true, needs_analysis=true

5. PROFILE_UPDATE
   Input: "My weight is now 75kg"
   Extracted: profile_field=weight, value=75

6. QUERY_HISTORY
   Input: "How much did I eat yesterday?"
   Extracted: query_type=food_history, time_range=yesterday

7. HELP_REQUEST
   Input: "What should I do?"
   Extracted: help_needed=true, context=general

8. SMALL_TALK
   Input: "How are you?"
   Extracted: social_interaction=true, response_type=greeting
```

#### Technology Stack for NLU:

| Technology | Approach | Accuracy | Setup Complexity |
|------------|----------|----------|------------------|
| **Rasa NLU** | Rule-based + ML | 85-92% | Medium |
| **Hugging Face Transformers** | Deep Learning | 88-95% | High |
| **spaCy** | Statistical NLP | 80-88% | Low-Medium |
| **BERT-based models** | Transfer Learning | 92-97% | High |
| **Custom ML Model** | Domain-specific | 90-95% | High |

#### Recommended: **Rasa NLU** with fallback to **spaCy** for resource efficiency

#### Entity Extraction:
```python
ENTITY_TYPES = {
    'FOOD_ITEM': {
        'examples': ['rice', 'chicken', 'bread', 'fruit'],
        'extraction': 'NER + food_ontology'
    },
    'QUANTITY': {
        'examples': ['1 cup', '200g', 'a plate'],
        'extraction': 'regex + context'
    },
    'TIME': {
        'examples': ['now', 'morning', 'yesterday', '3 hours ago'],
        'extraction': 'temporal NER'
    },
    'ACTIVITY': {
        'examples': ['walking', 'running', 'resting'],
        'extraction': 'domain_dictionary'
    },
    'HEALTH_METRIC': {
        'examples': ['glucose', 'weight', 'blood_pressure'],
        'extraction': 'domain_dictionary'
    }
}
```

#### Code-Switching Handling (Sinhala + English Mix):
```python
def handle_code_switching(text):
    """
    Handle: "I had rice and කුකුල්" (rice and chicken in Sinhala)
    
    Steps:
    1. Detect language boundaries
    2. Translate Sinhala components to English
    3. Process as single sentence
    4. Return unified understanding
    """
    
    text = "I had rice and කුකුල්"
    
    # Step 1: Language Detection
    language_map = detect_languages(text)
    # Result: [("en", "I had rice and"), ("si", "කුකුල්")]
    
    # Step 2: Component Translation
    sinhala_part = "කුකුල්" → translate → "chicken"
    
    # Step 3: Unified Processing
    unified_text = "I had rice and chicken"
    
    # Step 4: Extract Intent & Entities
    return {
        'intent': 'LOG_FOOD',
        'entities': {'food_items': ['rice', 'chicken']}
    }
```

---

### 2.3 Text Generation & Response Management
**Responsibility**: Generate natural, context-aware spoken responses

#### Response Strategy Matrix:

| Situation | Response Style | Example |
|-----------|-----------|----------|
| Food logging success | Encouraging + Tips | "Great! That meal has about 45g carbs. You might want to take a short walk." |
| Prediction alert | Alert + Action | "Your glucose may spike. Try drinking water and resting." |
| Clarification needed | Conversational | "Did you mean white rice or brown rice?" |
| Recommendation | Informative | "Based on your activity today, I suggest..." |
| Small talk | Friendly | "I'm doing well, thanks for asking!" |
| Error | Helpful | "I didn't quite get that. Could you repeat?" |

#### Response Generation Engine:
```python
class ResponseGenerator:
    def __init__(self, user_profile, system_context):
        self.user_profile = user_profile
        self.system_context = system_context
    
    def generate_response(self, intent, entities, context):
        # 1. Select response template based on intent
        # 2. Personalize based on user profile
        # 3. Add contextual recommendations
        # 4. Format for natural speech
        # 5. Apply tone adjustment
        
        response = {
            'text': "...",
            'tone': 'encouraging|neutral|warning',
            'confidence': 0.95,
            'follow_up_options': []
        }
        return response
```

#### Tone Personalization:
```python
TONE_PROFILES = {
    'encouraging': {  # For users who respond to positive reinforcement
        'phrases': ["Great job!", "That's excellent!", "Well done!"],
        'style': 'upbeat, motivating'
    },
    'neutral': {  # For task-oriented users
        'phrases': ["Logged", "Understood", "Processing..."],
        'style': 'direct, factual'
    },
    'cautious': {  # For users with poor glycemic control
        'phrases': ["Please note...", "Recommendation...", "Caution..."],
        'style': 'careful, alert'
    },
    'friendly': {  # For casual interaction
        'phrases': ["How's it going?", "Sounds good!", "Let's check..."],
        'style': 'conversational, warm'
    }
}
```

---

### 2.4 Text-to-Speech (TTS) Module
**Responsibility**: Convert generated text to natural-sounding speech

#### TTS Technology Options:

| Technology | Quality | Latency | Cost | Multilingual |
|-----------|---------|---------|------|-------------|
| **Google Cloud TTS** | Excellent | 200-400ms | Moderate | Yes |
| **Azure Speech TTS** | Excellent | 200-400ms | Low | Yes |
| **Coqui TTS** | Good | 500-1000ms | Free | Limited |
| **gTTS (Google)** | Good | 300-500ms | Free | Extensive |
| **Pyttsx3** | Fair | 100-200ms | Free | Limited |

#### Recommended: **Google Cloud TTS** primary, **Coqui TTS** fallback for offline

#### Voice Selection by Language:
```python
TTS_VOICES = {
    'english': {
        'provider': 'google',
        'voice_name': 'en-US-Neural2-C',  # Female voice
        'speaking_rate': 1.0,
        'pitch': 0.0
    },
    'sinhala': {
        'provider': 'google',
        'voice_name': 'si-LK-Standard-A',  # Available in Google v1
        'speaking_rate': 0.95,
        'pitch': 0.0
    },
    'tamil': {
        'provider': 'google',
        'voice_name': 'ta-IN-Standard-B',  # Female voice
        'speaking_rate': 0.95,
        'pitch': 0.0
    }
}
```

#### Audio Output Optimization:
```python
TTS_OPTIMIZATION = {
    'sample_rate': 24000,  # Higher for better quality
    'audio_format': 'MP3',  # Compressed for transmission
    'bit_depth': 16,
    'channels': 'mono',
    'volume_normalization': True,
    'speaking_rate_adjustment': {
        'fast_speech': 1.15,
        'normal_speech': 1.0,
        'clear_speech': 0.85
    }
}
```

---

## 3. Voice Input Pipeline (Complete Flow)

```
┌─────────────────────────────────────────────────────────────┐
│ USER SPEAKS                                                 │
│ "I had rice and chicken curry for lunch"                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ AUDIO CAPTURE & PREPROCESSING                               │
│ • Sample rate: 16kHz                                        │
│ • Format: PCM/WAV                                           │
│ • Noise filtering: Yes                                      │
│ • Silence detection: Yes                                    │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ SPEECH-TO-TEXT (STT)                                        │
│ Input: Audio bytes                                          │
│ Process: Speech recognition                                │
│ Output: "I had rice and chicken curry for lunch"           │
│ Confidence: 0.96                                            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ NATURAL LANGUAGE UNDERSTANDING (NLU)                        │
│ Intent: LOG_FOOD                                            │
│ Entities:                                                   │
│   • food_items: [rice, chicken, curry]                     │
│   • meal_type: lunch                                        │
│   • time: now                                               │
│ Confidence: 0.94                                            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ CONTEXT RETRIEVAL                                           │
│ • User profile: Type 2 Diabetes, needs careful carbs      │
│ • Today's stats: 1 meal logged, 3000 steps                │
│ • History: Usually logs 3 meals                            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ RESPONSE GENERATION                                         │
│ Template: Food logged + estimation + suggestion            │
│ Generated: "Logged your lunch. That's about 45g carbs.     │
│ You've walked 3000 steps, great! A 15-minute walk would   │
│ help balance those carbs."                                 │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ TEXT-TO-SPEECH (TTS)                                        │
│ • Language: English                                        │
│ • Voice: Female, natural                                   │
│ • Speed: Normal                                            │
│ Output: Audio MP3 stream                                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ AUDIO OUTPUT & FEEDBACK                                     │
│ Speaker plays: "Logged your lunch. That's about..."       │
│ User continues conversation or completes task              │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Conversation Management

### 4.1 Multi-Turn Conversation Handling
```python
class ConversationManager:
    """
    Maintains conversation context across multiple exchanges
    """
    
    def __init__(self, user_id):
        self.user_id = user_id
        self.conversation_history = []
        self.current_context = {}
        self.interaction_depth = 0
    
    def add_turn(self, user_input, ai_response, intent, entities):
        """Record each conversation turn"""
        turn = {
            'timestamp': now(),
            'user_input': user_input,
            'ai_response': ai_response,
            'intent': intent,
            'entities': entities,
            'context_used': self.current_context.copy()
        }
        self.conversation_history.append(turn)
    
    def get_conversation_context(self, depth=3):
        """Retrieve recent context for disambiguity"""
        recent_turns = self.conversation_history[-depth:]
        return {
            'recent_intents': [t['intent'] for t in recent_turns],
            'recent_entities': [t['entities'] for t in recent_turns],
            'topic': self._infer_topic(recent_turns)
        }
    
    def handle_clarification(self, user_input):
        """
        If NLU confidence < 0.7, ask for clarification
        
        Example:
        User: "I had spicy chicken"
        AI: "Did you mean fried chicken or curry chicken?"
        User: "Curry chicken"
        """
        options = self._generate_disambiguation_options()
        return options
```

### 4.2 Conversation State Machine
```
STATE: LISTENING
├─ Action: Capture audio, transcribe
├─ Transition: → UNDERSTANDING (on STT confidence > 0.7)
└─ Fallback: → CLARIFY (on low confidence)

STATE: UNDERSTANDING
├─ Action: Perform NLU, extract intent
├─ Transition: → RESPONDING (on intent confidence > 0.75)
└─ Fallback: → CLARIFY (on ambiguous intent)

STATE: CLARIFYING
├─ Action: Ask clarification question
├─ Transition: → UNDERSTANDING (on user response)
└─ Retry: → LISTENING (max 3 attempts)

STATE: RESPONDING
├─ Action: Generate response, convert to speech
├─ Transition: → LISTENING (ready for next input)
└─ Special: → PROCESSING (if action needed)

STATE: PROCESSING
├─ Action: Handle heavy computation (food analysis, prediction)
├─ Transition: → RESPONDING (with results)
└─ Notify: "One moment, let me check..."

STATE: ERROR
├─ Action: Fallback response
├─ Transition: → LISTENING (after error handling)
└─ Log: Error for debugging
```

---

## 5. Multilingual & Code-Switching Strategy

### 5.1 Language Detection:
```python
def detect_language(text):
    """
    Detect if input is English, Sinhala, Tamil, or mixture
    """
    from langdetect import detect
    from textblob import TextBlob
    
    language = detect(text)  # Returns: 'en', 'si', 'ta'
    
    # For code-switching (mixed language):
    if is_code_switched(text):
        return {
            'primary': 'sinhala',
            'secondary': 'english',
            'type': 'code-switched',
            'components': segment_by_language(text)
        }
```

### 5.2 Sinhala/Tamil Special Handling:
```python
SINHALA_CHALLENGES = {
    'challenge': 'Limited training data for speech recognition',
    'solution': 'Pre-built phoneme models + user-specific adaptation',
    'fallback': 'Ask for spelling or written input'
}

TAMIL_CHALLENGES = {
    'challenge': 'Diacritic complexity, morphological richness',
    'solution': 'Normalized processing, morphological analysis',
    'fallback': 'Voice to text confirmation'
}

CODE_SWITCHING_CHALLENGES = {
    'challenge': 'System doesn\'t know where language switches',
    'solution': 'Segment by language boundaries first, then process',
    'example': '"I had rice and කුකුල්" → ["I had rice and", "chicken"]'
}
```

---

## 6. Voice Session Management

### 6.1 Session Lifecycle:
```
WAKE_UP
│
├─ Trigger: "Hey Assistant" / App tap
├─ Duration: 30 seconds of silence → auto-close
├─ Timeout: 5 minutes inactive → close with goodbye
│
ACTIVE_CONVERSATION
│
├─ User can chain multiple requests
├─ Context preserved across turns
├─ Smart interruption handling (user says "wait" → pause)
│
BACKGROUND_MODE
│
├─ Optional: Continuous listening (battery intensive)
├─ Listen for keywords: "glucose alert", "meal alert"
├─ Low-power audio processing
│
SLEEP
│
└─ Clear memory, save conversation logs
```

### 6.2 Session Context:
```python
session_context = {
    'session_id': 'uuid',
    'user_id': 'user123',
    'start_time': '2024-03-22T10:30:00',
    'language': 'english',
    'interaction_count': 5,
    'topics_discussed': ['food_logging', 'activity', 'prediction'],
    'confidence_scores': [0.96, 0.94, 0.92],
    'errors_encountered': 0,
    'user_satisfaction': 'high'
}
```

---

## 7. Error Handling & Recovery

### 7.1 STT Errors:
```
Error: "Silent input"
Recovery: "I didn't hear that. Could you speak up?"

Error: "Noise interference"
Recovery: "It's a bit noisy. Can you move to a quieter place?"

Error: "Unsupported language"
Recovery: "I speak English, Sinhala, and Tamil. Which would you prefer?"

Error: "API timeout"
Recovery: "I'm having connection issues. Try again in a moment."
```

### 7.2 NLU Errors:
```
Error: "Ambiguous intent"
Recovery: "Did you mean A or B?"

Error: "Missing required entities"
Recovery: "How much did you eat? (in cups, grams, or bowls?)"

Error: "Out-of-domain request"
Recovery: "I'm specialized in diabetes management. Ask me about food, activity, or health!"
```

### 7.3 System Errors:
```
Error: "TTS unavailable"
Recovery: "I'll send you the message as text instead"
         + Display on minimal UI

Error: "Context engine timeout"
Recovery: "Let me log that. Recommendations coming in a moment..."
         + Proceed without personalized recommendations

Error: "Prediction model error"
Recovery: "I couldn't predict your glucose. Let's check your history instead."
```

---

## 8. Performance Metrics & Monitoring

### 8.1 Key Metrics:
```python
METRICS = {
    'speech_recognition': {
        'accuracy': '% of correctly transcribed words',
        'language_accuracy': {'english': 95, 'sinhala': 88, 'tamil': 85},
        'latency': 'ms from speech end to text',
        'timeout_rate': '% of failed recognitions'
    },
    'nlu': {
        'intent_accuracy': '% of correctly identified intents',
        'entity_extraction_f1': 'precision × recall balance',
        'clarification_rate': '% requiring clarification'
    },
    'tts': {
        'latency': 'ms from text to audio',
        'naturalness_score': 'MOS (Mean Opinion Score)',
        'error_rate': '% of failed generations'
    },
    'conversation': {
        'turn_latency': 'total ms per conversation turn',
        'user_satisfaction': 'survey score',
        'task_completion_rate': '% of requests fulfilled'
    }
}
```

### 8.2 Monitoring & Logging:
```python
logger.info({
    'event': 'conversation_turn',
    'user_id': 'user123',
    'stt_confidence': 0.96,
    'intent': 'LOG_FOOD',
    'nlu_confidence': 0.94,
    'response_latency_ms': 450,
    'tts_latency_ms': 280,
    'total_turn_latency_ms': 1200,
    'timestamp': '2024-03-22T10:30:45'
})
```

---

## 9. Security & Privacy in Voice Layer

### 9.1 Audio Privacy:
```python
PRIVACY_MEASURES = {
    'local_processing': {
        'stt_model': 'whisper-small runs locally if possible',
        'tts_model': 'coqui-tts for offline mode',
        'benefit': 'Audio never leaves device'
    },
    'audio_encryption': {
        'transport': 'TLS 1.3',
        'storage': 'AES-256 for audio logs',
        'retention': '30 days then delete'
    },
    'user_control': {
        'record_audio': 'user can disable logging',
        'delete_history': 'clear conversations anytime',
        'opt_out_improvements': 'don\'t use data for model training'
    }
}
```

### 9.2 Sensitive Information Handling:
```python
SENSITIVE_TERMS = [
    'blood_glucose_level',
    'medication_name',
    'weight',
    'health_condition'
]

# Don't store/log sensitive values in plain text
# Use: value_hash, temporal_reference, or aggregation
```

---

## 10. Implementation Checklist

- [ ] **STT Integration**
  - [ ] Google Speech-to-Text API setup
  - [ ] Whisper model fine-tuning for Sinhala/Tamil
  - [ ] Audio preprocessing pipeline
  - [ ] Confidence filtering

- [ ] **NLU Pipeline**
  - [ ] Rasa NLU setup with domain data
  - [ ] Intent training data (minimum 100 examples per intent)
  - [ ] Entity extraction models
  - [ ] Code-switching handler

- [ ] **Response Generation**
  - [ ] Template library for all intents
  - [ ] Personalization engine
  - [ ] Tone adaptation
  - [ ] Follow-up suggestion logic

- [ ] **TTS Integration**
  - [ ] Google Cloud TTS setup
  - [ ] Voice selection by language
  - [ ] Audio optimization pipeline
  - [ ] Fallback TTS system

- [ ] **Conversation Management**
  - [ ] Session storage
  - [ ] Context tracking
  - [ ] Multi-turn logic
  - [ ] Clarification handling

- [ ] **Testing & Validation**
  - [ ] STT accuracy tests (>90% for English)
  - [ ] Intent recognition tests
  - [ ] End-to-end conversation tests
  - [ ] Multilingual testing

---

## 11. Technology Stack Summary

```
┌─────────────────────────────────────────────────────────────┐
│ VOICE INTERFACE LAYER                                       │
├─────────────────────────────────────────────────────────────┤
│ STT          │ Google Speech API / Whisper / Vosk           │
│ NLU          │ Rasa NLU / spaCy / Transformers             │
│ Entity Ext.  │ spaCy NER / custom models                   │
│ TTS          │ Google Cloud TTS / Coqui TTS                │
│ Orchestration│ Python FastAPI + async handlers             │
│ Storage      │ Redis (session cache) + PostgreSQL (logs)   │
│ Monitoring   │ Prometheus + Grafana + ELK Stack            │
└─────────────────────────────────────────────────────────────┘
```

---

## 12. References & Appendices

### A. NLU Training Data Format (Rasa):
```yaml
nlu:
  - intent: log_food
    examples: |
      - I had rice and curry
      - I ate [chicken](FOOD_ITEM)
      - I just had [sandwich](FOOD_ITEM) for [lunch](MEAL_TIME)
      
  - intent: log_activity
    examples: |
      - I walked for [30 minutes](DURATION)
      - [Running](ACTIVITY) for [1 hour](DURATION)
```

### B. Response Templates:
```yaml
responses:
  utter_log_food_success:
    - text: "Great! I've logged {food_name}. That's approximately {calories} calories and {carbs}g carbs."
      
  utter_activity_logged:
    - text: "Excellent! {duration} minutes of {activity} burns about {calories} calories. Keep it up!"
```

---

**Last Updated**: March 22, 2026
**Status**: Complete
**Next Module**: 05_CONTEXT_ENGINE_DETAILED.md
