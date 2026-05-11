# 📱 Repo 03 — `glucovision-mobile`

> **Status:** Existing · Expand  
> **Flag:** —  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 6 (after auth/user/gateway)

---

## Purpose

The **primary patient-facing application** of GlucoVision. This Flutter app is how diabetic patients interact with the entire platform — logging food, tracking glucose, receiving recommendations, talking to the AI assistant, and viewing their digital twin health projections. Everything the backend computes surfaces here.

---

## Core Functionality

| Module | What It Does |
|---|---|
| Auth Module | Login, registration, JWT token management, biometric unlock |
| Food Logger | Camera-based food capture → sends to `09` recognition + `10` portion estimation |
| AR Portion Overlay | Augmented reality overlay showing estimated portion size from `10` |
| Glucose Tracker | Manual glucose entry + CGM sync display (from `14`) |
| BLE Wearable Sync | Connects to smart bands via Bluetooth LE (through `17`) |
| Recommendation Feed | Personalized meal and activity recommendations from `15` |
| AI Assistant | Voice and vision Q&A interface connecting to `16` |
| Digital Twin View | Interactive health projection charts from `18` |
| Notifications | FCM push notification receiver from `07` |
| Risk Alerts | Real-time glucose alert display from `13` via WebSocket |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Food photos | Device camera | JPEG |
| Voice commands | Device microphone | PCM audio → Whisper via `16` |
| Manual glucose readings | User entry | Float (mmol/L) |
| BLE data (heart rate, steps) | Smart band | BLE GATT packets via `17` |
| Push notifications | FCM via `07` | FCM payload |
| Recommendation data | `15` | JSON REST |
| Glucose predictions | `12` | JSON REST |
| Digital twin projections | `18` | JSON REST |

---

## Outputs

| Output | Destination | Format |
|---|---|---|
| Food image + metadata | `09` food recognition, `10` portion estimation | Multipart REST |
| Manual glucose log | `12` glucose prediction service | JSON REST |
| Activity log | `15` recommendation engine | JSON REST |
| User preferences | `06` user service | JSON REST |
| Auth credentials | `05` auth service | JSON REST / OAuth2 |

---

## Dependencies

| Service | Role |
|---|---|
| `05` auth-service | JWT tokens, user authentication |
| `06` user-service | User profile, preferences, health metadata |
| `07` notification-service | Push notifications via FCM |
| `08` api-gateway | All backend routing (single entry point) |
| `09` food-recognition | Food classification from photos |
| `10` portion-estimation | Portion size from camera |
| `12` glucose-prediction | Glucose forecast data |
| `13` risk-alert-engine | Real-time glucose alerts |
| `14` cgm-integration | CGM device data |
| `15` recommendation-engine | Diet and activity recommendations |
| `16` assistant-service | Voice/vision AI chat |
| `17` edge-iot-service | BLE wearable data bridge |
| `18` digital-twin | Health simulation projections |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | All data fetching from backend (via gateway 08) |
| WebSocket | Real-time glucose alerts from `13`, cooking guidance from `11` |
| Bluetooth LE (BLE) | Wearable smart band connection |
| FCM (Firebase Cloud Messaging) | Push notifications |
| Multipart/form-data | Food image uploads |
| WebRTC (optional) | AR camera stream to `10` portion estimator |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Flutter (Dart) |
| State Management | Riverpod / Bloc |
| Navigation | GoRouter |
| HTTP Client | Dio |
| WebSocket | dart:io WebSocket / web_socket_channel |
| BLE | flutter_blue_plus |
| AR | ARCore (Android) / ARKit (iOS) via arcore_flutter_plugin |
| Camera | camera plugin |
| Push Notifications | firebase_messaging |
| Local Storage | Hive / shared_preferences |
| Charts | fl_chart |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| On-device inference | TFLite food recognition (via `17` edge inference module) for offline fallback |
| AR geometry | Receives 3D overlay data from `10`, renders via ARCore/ARKit |
| Voice processing | Streams audio to `16` Whisper, receives text response |
| Prediction display | Shows glucose forecast curves from `12` (LSTM + Transformer ensemble) |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| Hive (local) | Offline food log cache, glucose readings | Key-Value |
| SharedPreferences | Auth tokens, user settings | Key-Value |
| Backend (via REST) | All persistent health records | JSON REST → PostgreSQL |
| InfluxDB (via `12`) | Glucose time-series (accessed read-only via REST) | Time-series |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Platforms | Android (primary) + iOS |
| Build | Flutter build pipeline (GitHub Actions) |
| Distribution | Google Play Store / TestFlight |
| Deep linking | GoRouter URI-based deep links |
| Background tasks | WorkManager for BLE sync + glucose polling |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Token storage | Secure storage (flutter_secure_storage) — never SharedPreferences for JWT |
| Biometric auth | LocalAuthentication plugin (fingerprint / face ID) |
| HTTPS enforcement | Certificate pinning for medical data endpoints |
| Sensitive data | Glucose + nutrition data never logged to console |
| Permission model | Camera, microphone, BLE — all with runtime permission checks |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Image upload | Compress images before upload (reduce bandwidth) |
| BLE polling | Throttle BLE reads to avoid battery drain |
| Real-time alerts | WebSocket reconnect with exponential backoff |
| Chart rendering | Lazy load glucose history; paginate large datasets |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Bloc/Riverpod state logic, data parsing |
| Widget Tests | Key screens (food logger, glucose chart) |
| Integration Tests | End-to-end flows (login → capture food → view recommendation) |
| Device Testing | Android emulator + physical device for BLE/camera |

---

## Observability

| Concern | Detail |
|---|---|
| Crash reporting | Firebase Crashlytics |
| Analytics | Firebase Analytics (screen views, feature usage) |
| Performance | Firebase Performance Monitoring (API latency, startup time) |
| Error logging | Structured error capture with user context (no PII in logs) |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟡 Intermediate | Multi-protocol integration (BLE, WebSocket, REST, AR), complex state management across many health data streams |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Production Flutter app with complex multi-service integration |
| AR portion overlay implementation |
| BLE wearable integration |
| Real-time WebSocket health alert UI |
| AI-powered voice + vision assistant interface |
| Offline-first health data caching |
| Firebase services integration (FCM, Crashlytics, Analytics) |
