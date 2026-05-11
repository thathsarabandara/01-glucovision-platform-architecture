# 📚 GlucoVision — Repo Knowledge Index

> All 21 repository knowledge documents for the GlucoVision AI-Powered Diabetic Management Platform.  
> Based on: *Systematic Review of AI-Based PDAR System for Diabetic Individuals*

---

## Quick Reference Table

| # | Repo | Flag | Stack | Difficulty | Build Priority |
|---|---|---|---|---|---|
| 01 | [platform-architecture](./01-platform-architecture.md) | — | Markdown / OpenAPI | ⭐ | 1 |
| 02 | [data-collect-app](./02-data-collect-app.md) | — | Flutter + FastAPI + React | ⭐⭐ | 2 |
| 03 | [glucovision-mobile](./03-glucovision-mobile.md) | — | Flutter | ⭐⭐ | 6 |
| 04 | [web-dashboard](./04-web-dashboard.md) | — | React + Vite | ⭐⭐ | 7 |
| 05 | [auth-service](./05-auth-service.md) | 🔐 Security Critical | FastAPI + Redis + PostgreSQL | ⭐⭐ | 3 |
| 06 | [user-service](./06-user-service.md) | — | FastAPI + PostgreSQL | ⭐⭐ | 4 |
| 07 | [notification-service](./07-notification-service.md) | — | FastAPI + Celery + FCM | ⭐⭐ | 8 |
| 08 | [api-gateway](./08-api-gateway.md) | — | Traefik + Prometheus + Grafana | ⭐⭐ | 5 |
| 09 | [food-recognition-service](./09-food-recognition-service.md) | GPU | PyTorch + ResNet/ViT + ONNX | ⭐⭐⭐ | 9 |
| 10 | [portion-estimation-service](./10-portion-estimation-service.md) | GPU | PyTorch + Mask R-CNN + Open3D | ⭐⭐⭐ | 12 |
| 11 | [cooking-ai-service](./11-cooking-ai-service.md) | GPU | PyTorch + 3D-CNN + WebSocket | ⭐⭐⭐⭐ | 15 |
| 12 | [glucose-prediction-service](./12-glucose-prediction-service.md) | 🔴 Medical Critical | PyTorch + LSTM + Transformer + InfluxDB | ⭐⭐⭐⭐ | 13 |
| 13 | [risk-alert-engine](./13-risk-alert-engine.md) | 🔴 Safety Critical | FastAPI + Kafka + Redis | ⭐⭐⭐⭐ | 14 |
| 14 | [cgm-integration-service](./14-cgm-integration-service.md) | ⚡ Real-Time | FastAPI + Celery + InfluxDB | ⭐⭐⭐ | 10 |
| 15 | [recommendation-engine](./15-recommendation-engine.md) | — | FastAPI + LLaMA-3 + LangChain + RL | ⭐⭐⭐⭐ | 16 |
| 16 | [assistant-service](./16-assistant-service.md) | — | Whisper + LLaMA-3 Vision + TTS | ⭐⭐⭐⭐ | 17 |
| 17 | [esp32-cam-streaming-service](./17-esp32-cam-streaming-service.md) | — | C++ (ESP-IDF) + aiortc + WebRTC + MQTT | ⭐⭐⭐ | 11 |
| 18 | [wearable-sync-service](./18-wearable-sync-service.md) | — | Flutter BLE + Python bleak + InfluxDB | ⭐⭐⭐ | 11 |
| 19 | [digital-twin](./19-digital-twin.md) | 🔬 Research | torchdiffeq + Neural ODE + InfluxDB | ⭐⭐⭐⭐⭐ | 18 |
| 20 | [federated-learning](./20-federated-learning.md) | 🔬 Privacy Critical | Flower + Opacus + PySyft | ⭐⭐⭐⭐⭐ | 19 |
| 21 | [devops-infra](./21-devops-infra.md) | — | K8s + Helm + Terraform + GitHub Actions | ⭐⭐⭐⭐ | 20 |

> **Note:** The `18-digital-twin.md`, `19-federated-learning.md`, and `20-devops-infra.md` legacy files are superseded by `19-digital-twin.md`, `20-federated-learning.md`, and `21-devops-infra.md` respectively following the IoT split.

---

## Architecture Groups

### 🏛️ Existing Repos (4)
- `01` Platform Architecture — design hub
- `02` Data Collection App — Sri Lankan food dataset pipeline
- `03` GlucoVision Mobile — patient Flutter app
- `04` Web Dashboard — clinician/researcher React portal

### 🔧 Core Backend (4)
- `05` Auth Service 🔐 — JWT + OAuth2 + RBAC + MFA
- `06` User Service — patient/clinician profiles
- `07` Notification Service — FCM/APNs push alerts
- `08` API Gateway — Traefik + full observability stack

### 🍽️ Food AI Layer (3)
- `09` Food Recognition — ResNet-50 / ViT classification
- `10` Portion Estimation — Mask R-CNN + depth + 3D volume
- `11` Cooking AI — 3D-CNN temporal cooking state

### 🩺 Health AI — Medical Critical (3)
- `12` Glucose Prediction 🔴 — Bi-LSTM + Transformer ensemble
- `13` Risk Alert Engine 🔴 — Kafka-based real-time alerts
- `14` CGM Integration ⚡ — Dexcom + FreeStyle Libre APIs

### 🧠 Intelligence Layer (2)
- `15` Recommendation Engine — GI diet + RL + LLaMA-3 advisory
- `16` Assistant Service — Whisper + LLaMA-3 Vision + TTS

### 🔌 IoT & Edge (2) ← *split from original single repo*
- `17` ESP32-CAM Streaming — firmware (C++ ESP-IDF) + WebRTC + TFLite edge inference
- `18` Wearable Sync — BLE GATT smart band + activity relay + InfluxDB

### 🔬 Research Grade (2)
- `19` Digital Twin 🔬 — Neural ODE physiological simulation
- `20` Federated Learning 🔬 — Flower + ε-DP + SecAgg

### 🏗️ Infrastructure (1)
- `21` DevOps Infra — K8s + Terraform + CI/CD for all 21 services

---

## IoT Split Rationale

| Concern | `17` ESP32-CAM Streaming | `18` Wearable Sync |
|---|---|---|
| Hardware | ESP32-CAM (camera) | Smart band (BLE peripheral) |
| Protocol | WiFi + MJPEG + WebRTC | Bluetooth LE (GATT) |
| Language | C++ (ESP-IDF firmware) | Flutter BLE plugin / Python bleak |
| Data type | Video frames → `09`, `11` | HR, steps, SpO2 → `15`, `12` |
| Failure mode | WiFi drop, OTA failure | BLE disconnect, GATT timeout |
| Scaling | Stream relay instances | BLE polling worker instances |

---

## Data Store Ownership

| Store | Owner Service | What It Stores |
|---|---|---|
| PostgreSQL | `05`, `06`, `07`, `09`, `15`, `17`, `18` | User data, profiles, food records, device registry |
| InfluxDB | `12`, `14`, `18`, `19` | Glucose time-series, wearable activity, twin state |
| Redis | `05`, `07`, `08`, `13`, `19` | Token blacklist, cache, rate limits, alert state |
| MinIO / S3 | `02`, `09`, `10`, `16`, `17` | Images, model weights, audio, OTA firmware |
| MLflow | `09`, `12`, `20` | Model registry, experiment tracking |
| Kafka | `13` | Alert event stream |
| ChromaDB / pgvector | `15` | Nutrition RAG knowledge base |
| MQTT Broker | `17` | ESP32-CAM telemetry + commands |

---

## Critical Warnings

> [!CAUTION]
> **`12` and `13` must NEVER share a deployment unit.** Glucose prediction and risk alerts require independent rollback. A bad model deploy in `12` must not take down `13` alerts.

> [!WARNING]
> **`05` auth-service builds first** — every other service depends on it. Build and test it in full isolation before wiring any other service.

> [!IMPORTANT]
> **`17` and `18` build in parallel** — they are independent (different hardware, different protocols). No dependency between them.

> [!NOTE]
> **`19` digital-twin and `20` federated-learning** are research differentiators. Plan their InfluxDB schema and gradient format contracts early even if you build them last.
