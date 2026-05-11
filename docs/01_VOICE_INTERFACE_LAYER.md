# 01. Voice Interface Layer - Complete Architecture & Implementation

## 📋 Overview
The Voice Interface Layer is the primary user interaction point, handling real-time speech processing and natural language understanding. This layer enables hands-free, voice-first interaction without requiring traditional UI navigation.

---

## 🏗️ Component Architecture

### 1.1 Speech-to-Text (STT) Module

#### Purpose
Convert user speech into text for processing and understanding.

#### Technology Options

| Component | Technology | Pros | Cons | Use Case |
|-----------|-----------|------|------|----------|
| **Cloud-Based** | Google Cloud Speech-to-Text, Azure Speech | High accuracy (95%+), Multiple language support | Requires internet, Privacy concerns, Latency | Primary choice for accuracy |
| **Edge/Offline** | Vosk, CMU Sphinx | Privacy-focused, No latency, Offline capable | Lower accuracy, Limited language support | Low-resource environments, Offline scenarios |
| **Premium Hybrid** | OpenAI Whisper | High accuracy, Multilingual, Open-source option | Moderate compute requirements | Best balance for this project |

#### Recommended Stack: **OpenAI Whisper + Vosk Fallback**

**Whisper Configuration for Multilingual Support:**
```python
import whisper

# Load Whisper model (small/medium/large)
model = whisper.load_model("base")  # English + Sinhala + Tamil capable

# Optimized parameters for Sinhala/Tamil
config = {
    "language": "si",  # Sinhala
    "temperature": 0.2,  # Lower for consistency
    "best_of": 3,  # Multiple attempts
    "beam_size": 5  # Better accuracy
}

# Process audio
result = model.transcribe("audio.wav", **config)
transcript = result["text"]
```

**Audio Input Handling:**
```python
import pyaudio
import numpy as np
import soundfile as sf

class AudioCapture:
    def __init__(self, sample_rate=16000, chunk_size=1024):
        self.sample_rate = sample_rate
        self.chunk_size = chunk_size
        self.audio = pyaudio.PyAudio()
    
    def capture_audio(self, duration=10, threshold=1500):
        """
        Capture audio with voice activity detection (VAD)
        """
        stream = self.audio.open(
            format=pyaudio.paFloat32,
            channels=1,
            rate=self.sample_rate,
            input=True,
            frames_per_buffer=self.chunk_size
        )
        
        frames = []
        silent_chunks = 0
        
        while True:
            data = np.frombuffer(stream.read(self.chunk_size), dtype=np.float32)
            frames.append(data)
            
            # Voice Activity Detection
            amplitude = np.abs(data).mean() * 32768
            
            if amplitude < threshold:
                silent_chunks += 1
            else:
                silent_chunks = 0
            
            # Stop if silence > 2 seconds
            if silent_chunks > (2 * self.sample_rate // self.chunk_size):
                break
        
        stream.stop_stream()
        stream.close()
        
        audio_data = np.concatenate(frames)
        return audio_data
```

### 1.2 Natural Language Understanding (NLU) Module

#### Purpose
Parse user intent and extract relevant entities (meals, activities, symptoms, etc.)

#### Intent Classification Framework

**Supported Intent Categories:**

```yaml
Intents:
  Profile_Management:
    - SetAge: "I am 35 years old"
    - SetWeight: "My weight is 75 kg"
    - SetDiabetesType: "I have type 2 diabetes"
    
  Food_Logging:
    - LogMeal: "I ate rice and curry"
    - LogSnack: "I had an apple"
    - LogDrink: "I drank orange juice"
    
  Activity_Tracking:
    - LogExercise: "I walked for 30 minutes"
    - LogActivity: "I did yoga"
    
  Health_Inquiry:
    - AskGlucose: "What's my glucose level?"
    - AskRecommendation: "What should I eat?"
    
  Symptom_Report:
    - ReportSymptom: "I feel dizzy"
    - ReportThirst: "I'm very thirsty"
```

#### NLU Implementation (Using Rasa/spaCy)

```python
import spacy
from rasa.nlu.model import Interpreter

class NLUProcessor:
    def __init__(self, language="si"):
        """Initialize NLU with multilingual support"""
        self.language = language
        self.rasa_interpreter = Interpreter.load("models/nlu")
        
        # Language-specific models
        self.spacy_models = {
            "si": spacy.load("si_core_news_sm"),  # Sinhala
            "ta": spacy.load("ta_core_news_sm"),  # Tamil
            "en": spacy.load("en_core_web_sm")    # English
        }
    
    def extract_intent(self, text):
        """
        Extract intent and confidence score
        """
        result = self.rasa_interpreter.parse(text)
        
        intent = result["intent"]["name"]
        confidence = result["intent"]["confidence"]
        
        return {
            "intent": intent,
            "confidence": confidence,
            "entities": self._extract_entities(text, intent)
        }
    
    def _extract_entities(self, text, intent):
        """
        Extract named entities based on context
        """
        nlp = self.spacy_models[self.language]
        doc = nlp(text)
        
        entities = {
            "food_items": [],
            "quantities": [],
            "durations": [],
            "health_metrics": [],
            "symptoms": []
        }
        
        for ent in doc.ents:
            if ent.label_ in ["FOOD", "PRODUCT"]:
                entities["food_items"].append(ent.text)
            elif ent.label_ == "QUANTITY":
                entities["quantities"].append(ent.text)
            elif ent.label_ == "DATE":
                entities["durations"].append(ent.text)
        
        return entities
    
    def parse_meal_description(self, text):
        """
        Parse detailed meal information from natural language
        Example: "I ate one cup of rice with chicken curry and one glass of water"
        """
        meal_info = {
            "items": [],
            "quantities": [],
            "total_calories": 0,
            "macronutrients": {},
            "confidence": 0.0
        }
        
        # Use regex patterns for common phrases
        import re
        
        patterns = {
            "quantity": r"(\d+\.?\d*)\s*(cup|bowl|spoon|gram|g|ml|litre|l)",
            "food": r"(rice|curry|bread|milk|fruit|vegetable)",
            "drink": r"(water|juice|coffee|tea|milk)"
        }
        
        for pattern_type, pattern in patterns.items():
            matches = re.finditer(pattern, text, re.IGNORECASE)
            for match in matches:
                meal_info["items"].append({
                    "name": match.group(1),
                    "type": pattern_type,
                    "quantity": match.group(1) if pattern_type == "quantity" else None
                })
        
        return meal_info
```

#### Multilingual Intent Matching

```python
class MultilingualNLU:
    def __init__(self):
        self.language_map = {
            "si": "sinhala",
            "ta": "tamil",
            "en": "english"
        }
        
        self.intent_translations = {
            "LogMeal": {
                "en": ["i ate", "i had", "i consumed"],
                "si": ["මම කෑවා", "මම ගත්තා"],
                "ta": ["நான் சாப்பிட்டேன்", "நான் வாங்கினேன்"]
            },
            "LogExercise": {
                "en": ["i walked", "i exercised", "i worked out"],
                "si": ["මම ඉවිදා ගියා", "මම ව්‍යාયාම කළා"],
                "ta": ["நான் நடந்தேன்", "நான் உடற்பயிற்சி செய்தேன்"]
            }
        }
    
    def detect_language(self, text):
        """Auto-detect language using langdetect"""
        from langdetect import detect
        
        lang_code = detect(text)
        return lang_code if lang_code in self.language_map else "en"
    
    def parse_multilingual(self, text):
        """Parse text in any supported language"""
        detected_lang = self.detect_language(text)
        
        # Standard NLU processing
        result = {
            "original_language": detected_lang,
            "translated_to_english": self._translate(text, detected_lang),
            "intent": self._classify_intent(text, detected_lang),
            "entities": self._extract_entities(text, detected_lang)
        }
        
        return result
    
    def _translate(self, text, src_lang):
        """Translate to English for processing"""
        if src_lang == "en":
            return text
        
        from google.cloud import translate_v2
        client = translate_v2.Client()
        
        result = client.translate_text(text, source_language=src_lang, target_language='en')
        return result['translatedText']
```

### 1.3 Text-to-Speech (TTS) Module

#### Purpose
Generate natural-sounding voice responses

#### Technology Stack

| Component | Provider | Quality | Latency | Multi-lang | Cost |
|-----------|----------|---------|---------|-----------|------|
| **Cloud** | Google Cloud TTS | Excellent | 200-500ms | Yes | Medium |
| **Edge** | Coqui TTS (Open-source) | Good | 300-800ms | Limited | Free |
| **Hybrid** | Azure TTS | Excellent | 150-300ms | Yes | Medium |

#### Recommended: **Azure TTS + Coqui Fallback**

```python
import azure.cognitiveservices.speech as speechsdk
from TTS.api import TTS

class TTSProcessor:
    def __init__(self, api_key, region, language="si"):
        self.language = language
        
        # Azure Speech Config
        speech_config = speechsdk.SpeechConfig(
            subscription=api_key,
            region=region
        )
        
        # Set voice based on language
        self.voices = {
            "si": "si-LK-SamithNeural",  # Sinhala
            "ta": "ta-IN-SarathNeural",   # Tamil
            "en": "en-US-AriaNeural"      # English
        }
        
        speech_config.speech_synthesis_voice_name = self.voices[language]
        self.speech_config = speech_config
        
        # Coqui TTS fallback (offline)
        self.tts_model = TTS(gpu=True, model_name='tts_models/mul_dataset/your_tts_mi')
    
    def synthesize_speech(self, text):
        """
        Synthesize text to speech with cloud provider
        Falls back to local TTS if cloud fails
        """
        try:
            # Try Azure first
            audio_output = speechsdk.AudioOutputConfig(use_default_speaker=True)
            synthesizer = speechsdk.SpeechSynthesizer(
                speech_config=self.speech_config,
                audio_config=audio_output
            )
            
            result = synthesizer.speak_text_async(text).get()
            
            if result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
                return result.audio_data
            else:
                # Fallback to local
                return self._synthesize_local(text)
                
        except Exception as e:
            print(f"Cloud TTS failed: {e}. Using local TTS.")
            return self._synthesize_local(text)
    
    def _synthesize_local(self, text):
        """
        Local TTS using Coqui (offline capability)
        """
        output_path = "/tmp/tts_output.wav"
        self.tts_model.tts_to_file(
            text=text,
            file_path=output_path,
            language=self.language
        )
        
        with open(output_path, 'rb') as f:
            return f.read()
    
    def adjust_synthesis_parameters(self, text, user_preferences):
        """
        Personalize speech output based on user preferences
        """
        ssml = self._text_to_ssml(text, user_preferences)
        
        # Use SSML for fine control
        audio_output = speechsdk.AudioOutputConfig(use_default_speaker=True)
        synthesizer = speechsdk.SpeechSynthesizer(
            speech_config=self.speech_config,
            audio_config=audio_output
        )
        
        result = synthesizer.speak_ssml_async(ssml).get()
        return result.audio_data
    
    def _text_to_ssml(self, text, preferences):
        """
        Convert text to SSML for fine control
        """
        rate = preferences.get("speech_rate", 1.0)
        pitch = preferences.get("pitch", "default")
        
        ssml = f"""
        <speak>
            <voice name="{self.voices[self.language]}">
                <prosody rate="{rate}" pitch="{pitch}">
                    {text}
                </prosody>
            </voice>
        </speak>
        """
        return ssml
```

### Personalized Voice Control

```python
class VoicePersonalization:
    def __init__(self, user_id):
        self.user_id = user_id
        self.preferences = self._load_preferences()
    
    def _load_preferences(self):
        """Load user voice preferences"""
        return {
            "preferred_language": "si",
            "speech_rate": 1.1,  # Slightly faster
            "pitch": "medium",
            "voice_gender": "neutral",
            "formality_level": "casual",  # Casual vs formal
            "response_style": "concise"    # Concise vs detailed
        }
    
    def personalize_response(self, response_text, context):
        """
        Adjust response based on user context
        - Time of day
        - User mood/stress level (detected from voice)
        - Previous interaction style
        """
        if context.get("hour") < 6:
            # Morning: encouraging tone
            prefix = "Good morning! "
        elif context.get("stress_level") == "high":
            # Calm, supportive tone
            response_text = response_text.replace(
                "must", "consider"
            )
        
        return response_text
```

---

## 📊 Data Flow

```
User Speech 
    ↓
[Voice Activity Detection (VAD)]
    ↓
[Audio Preprocessing]
    ↓
[Whisper STT]
    ↓
[Language Detection]
    ↓
[Multilingual NLU]
    ↓
[Intent Classification + Entity Extraction]
    ↓
[Context Engine]
    ↓
[Response Generation]
    ↓
[TTS Synthesis]
    ↓
User hears response
```

---

## 🔧 Implementation Roadmap

### Phase 1: Basic Voice I/O (Week 1-2)
- [ ] Implement Whisper STT with Sinhala support
- [ ] Basic pyaudio input setup
- [ ] Azure TTS integration
- [ ] Test accuracy on 100 audio samples

### Phase 2: Intent Understanding (Week 3-4)
- [ ] Rasa NLU model training
- [ ] Multilingual intent dataset creation
- [ ] Entity extraction pipeline
- [ ] Test 20+ intent types

### Phase 3: Personalization (Week 5)
- [ ] User preference storage
- [ ] Voice characteristic learning
- [ ] Response personalization

---

## 📈 Performance Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| STT Accuracy | > 90% | WER (Word Error Rate) |
| Intent Accuracy | > 85% | Classification accuracy |
| TTS Latency | < 500ms | Time from text → audio |
| End-to-End Response | < 2s | Speech in to speech out |

---

## 🚀 Advanced Features

### Background Always-On Mode
```python
class AlwaysOnAssistant:
    def __init__(self, hotword="assistant"):
        self.hotword = hotword
        self.listening = False
    
    def background_listen(self):
        """
        Continuously listen for hotword without consuming power
        Uses low-level keyword spotting
        """
        try:
            from porcupine import Porcupine
            
            porcupine = Porcupine(
                access_key="YOUR_ACCESS_KEY",
                keywords=[self.hotword]
            )
            
            while True:
                pcm = recorder.read()
                keyword_index = porcupine.process(pcm)
                
                if keyword_index >= 0:
                    self.listening = True
                    self._start_recording()
        
        except Exception as e:
            print(f"Hotword detection error: {e}")
```

### Emotion Detection from Voice
```python
class VoiceEmotionDetector:
    def detect_emotion(self, audio_file):
        """
        Detect user emotion from voice tone
        Affects response generation
        """
        import librosa
        import numpy as np
        
        y, sr = librosa.load(audio_file)
        
        # Extract audio features
        mfcc = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=13)
        zero_crossing = librosa.feature.zero_crossing_rate(y)
        
        # Emotion classification
        emotions = {
            "happy": mfcc.mean() > 0.5,
            "stressed": zero_crossing.mean() > 0.1,
            "tired": mfcc.std() < 0.3,
            "neutral": True  # default
        }
        
        return emotions
```

---

## 📚 Dependencies

```yaml
voice_interface_dependencies:
  STT:
    - openai-whisper>=20240101
    - librosa>=0.10.0
    - pyaudio>=0.2.13
    - numpy>=1.24.0
  
  NLU:
    - rasa>=3.5.0
    - spacy>=3.7.0
    - langdetect>=1.0.9
    - google-cloud-translate>=3.11.0
  
  TTS:
    - azure-cognitiveservices-speech>=1.31.0
    - coqui-tts>=0.14.0
    - pyttsx3>=2.70  # Fallback
  
  Emotion/Advanced:
    - porcupine-speech-recognition>=2.2.0
    - librosa>=0.10.0
    - tensorflow>=2.13.0
```

---

## 🔐 Privacy & Offline Capabilities

- **Whisper**: Can run fully offline (no API required)
- **Rasa NLU**: Custom models trained locally
- **Coqui TTS**: Offline-first approach
- **Audio Processing**: Done on-device, no cloud transmission except for optional cloud backup

---

## 📝 Notes

This layer is the primary entry point. All other systems depend on accurate intent extraction from this module.
