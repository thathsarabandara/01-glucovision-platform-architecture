# GlucoVision — 33-Repo Hybrid Architecture Plan

> Based on: *Systematic Review of AI-Based PDAR System for Diabetic Individuals*
> Strategy: Resource-efficient scaling via Always-On Core vs. On-Demand Serverless Tiers.

---

## 🔍 Hybrid Separation of Concerns

| Tier | Reason to Separate | Applies To |
|---|---|---|
| 🔴 **Tier 1: Always-On Core** | Medical safety, constant uptime, low latency routing | Gateways, Auth, Risk Engine, CGM sync, Live Video |
| 🟡 **Tier 2: On-Demand Serverless** | Resource optimization, Scale-to-Zero, event-triggered | AI Vision, Notification, User profiles, Nutrition engines |
| 🔵 **Tier 3: Async Batch** | Heavy compute, non-time-critical research | Digital Twin, Federated Learning |
| 📱 **Tier 0: Edge & Clients** | Device-side processing, UI rendering | Mobile, Web, ESP32 hardware |

---

## 📦 All 33 Repositories

```mermaid
graph TD
    subgraph EDGE["Tier 0: Edge (4)"]
        R03["03 mobile-app"]
        R04["04 web-dashboard"]
        R18["18 esp32-glasses"]
        R19["19 edge-streaming"]
    end

    subgraph CORE["Tier 1: Always-On Core (5)"]
        R05["05 api-gateway"]
        R06["06 auth-service 🔐"]
        R11["11 risk-engine 🔴"]
        R26["26 cgm-integration ⚡"]
        R02["02 system-orchestrator
(Event Bus)"]
    end

    subgraph ONDEMAND_API["Tier 2: On-Demand Backend (4)"]
        R07["07 user-service"]
        R08["08 notification-service"]
        R12["12 glucose-input-service"]
        R13["13 wearable-sync"]
    end

    subgraph ONDEMAND_AI["Tier 2: On-Demand AI Engines (14)"]
        R09["09 nutrition-engine"]
        R10["10 energy-engine"]
        R14["14 food-recognition"]
        R15["15 ingredient-detection"]
        R16["16 portion-estimation"]
        R17["17 cooking-state-ai"]
        R20["20 voice-assistant"]
        R21["21 vision-assistant"]
        R22["22 cooking-assistant"]
        R23["23 context-engine"]
        R24["24 glucose-prediction"]
        R25["25 diet-recommendation"]
        R27["27 personalization-engine"]
        R28["28 postmeal-analysis"]
    end

    subgraph RESEARCH["Tier 3: Async Batch (3)"]
        R29["29 digital-twin 🔬"]
        R30["30 simulation-engine"]
        R31["31 federated-learning 🔬"]
    end

    subgraph INFRA["Infrastructure (3)"]
        R01["01 platform-architecture"]
        R32["32 devops"]
        R33["33 observability"]
    end
```

---

## 🗂️ Repo Details

### `01` platform-architecture *(Existing)*
Docs, OpenAPI specs, Mermaid diagrams, ADRs, hybrid architecture contracts.

### `02` system-orchestrator *(MASTER BRAIN — Tier 1)*
**Why**: Defines system rules before services exist. Central event registry.
| Module | Features |
|---|---|
| `registry/` | Service discovery and routing logic |
| `event_bus/` | Kafka schema definitions |

### `03` mobile-app *(Edge)*
Flutter app: UI structure, camera flow, food logger, AR overlays.

### `04` web-dashboard *(Edge)*
React/Vite: Patient analytics, glucose charts, model monitoring.

---

### `05` api-gateway *(Tier 1: Always-On)*
Everything flows through this. Must be up 24/7.
| Module | Features |
|---|---|
| `gateway/` | Traefik routing, rate limiting |
Tech: Traefik

### `06` auth-service 🔐 *(Tier 1: Always-On)*
Security-critical identity service.
| Module | Features |
|---|---|
| `auth/` | JWT issue/refresh, OTP system |
Tech: FastAPI + Redis + PostgreSQL

### `07` user-service *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `profiles/` | Profiles, healthcare metadata |
Tech: FastAPI + PostgreSQL

### `08` notification-service *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `push/` | OTP emails, push alerts, reminders |
Tech: FastAPI + Celery + FCM

---

### `09` nutrition-engine *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `nutrition/` | Base logic for macros and diet AI |
Tech: FastAPI + PostgreSQL

### `10` energy-engine *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `energy/` | Calorie burn logic based on user data |
Tech: FastAPI

### `11` risk-engine 🔴 *(Tier 1: Always-On)*
Needs continuous uptime for hypo/hyper detection.
| Module | Features |
|---|---|
| `monitor/` | Real-time glucose thresholds |
Tech: FastAPI + Redis

### `12` glucose-input-service *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `input/` | Manual glucose entry API |
Tech: FastAPI

### `13` wearable-sync *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `sync/` | Syncs data from Samsung Fit/Health Connect |
Tech: FastAPI

---

### `14` food-recognition *(Tier 2: GPU)*
| Module | Model | Paper Ref |
|---|---|---|
| `classifier/` | ResNet-50 / ViT-B16 | [4][34] — 91–93% acc |
Tech: FastAPI + PyTorch

### `15` ingredient-detection *(Tier 2: GPU)*
| Module | Model | Paper Ref |
|---|---|---|
| `segmentation/`| Mask R-CNN | [5] |
Tech: FastAPI + PyTorch

### `16` portion-estimation *(Tier 2: GPU)*
| Module | Model | Paper Ref |
|---|---|---|
| `volume/` | RGB-D Depth Volume | [11][12] |
Tech: FastAPI + PyTorch + Open3D

### `17` cooking-state-ai *(Tier 2: GPU)*
| Module | Features |
|---|---|
| `state/` | 3D-CNN temporal logic |
Tech: FastAPI + PyTorch

---

### `18` esp32-glasses *(Edge)*
| Module | Features |
|---|---|
| `firmware/` | Hardware firmware base |
Tech: C++ (ESP-IDF)

### `19` edge-streaming *(Tier 1: Always-On)*
| Module | Features |
|---|---|
| `webrtc/` | Backbone for glasses + mobile streaming |
Tech: Python (aiortc)

### `20` voice-assistant *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `voice/` | Whisper ASR → intent |
Tech: FastAPI + Whisper

### `21` vision-assistant *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `vision/` | Combines camera + LLaMA-3 Vision |
Tech: FastAPI + LLaMA-3-Vision

### `22` cooking-assistant *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `assistant/` | Real-world AI helper |
Tech: FastAPI

---

### `23` context-engine *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `context/` | Central brain / Data fusion |
Tech: FastAPI

### `24` glucose-prediction *(Tier 2: Serverless)*
| Module | Model | Paper Ref |
|---|---|---|
| `forecast/` | LSTM + Transformer | [23][24] |
Tech: FastAPI + PyTorch

### `25` diet-recommendation *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `recommend/` | Personalized AI system |
Tech: FastAPI + LangChain

### `26` cgm-integration ⚡ *(Tier 1: Always-On)*
| Module | Features |
|---|---|
| `cgm/` | Polling Dexcom/Libre loops |
Tech: FastAPI + InfluxDB

---

### `27` personalization-engine *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `adapt/` | Behavior adaptation layer |
Tech: FastAPI + RL

### `28` postmeal-analysis *(Tier 2: Serverless)*
| Module | Features |
|---|---|
| `analysis/` | Feedback learning loop (2hrs post-meal) |
Tech: FastAPI

### `29` digital-twin 🔬 *(Tier 3: Batch)*
| Module | Features |
|---|---|
| `twin/` | Virtual metabolic model (Neural ODE) |
Tech: FastAPI + torchdiffeq

### `30` simulation-engine *(Tier 3: Batch)*
| Module | Features |
|---|---|
| `sim/` | What-if prediction system |
Tech: FastAPI

### `31` federated-learning 🔬 *(Tier 3: Batch)*
| Module | Features |
|---|---|
| `fl_server/` | Privacy-preserving distributed learning |
Tech: Flower (flwr)

---

### `32` devops *(Infra)*
| Module | Features |
|---|---|
| `k8s/` | CI/CD + deployment |

### `33` observability *(Infra)*
| Module | Features |
|---|---|
| `metrics/` | Prometheus, Grafana, Tracing |

---

## 🏗️ Full System Architecture (Detailed Hybrid Topology)

```mermaid
graph TB
    subgraph EDGE["📱 Tier 0: Edge Layer"]
        MOB["📱 Mobile App (03)
Flutter"]
        WEB["🖥️ Web Dashboard (04)
React/Vite"]
        GLASSES["🥽 ESP32 Glasses (18)"]
    end

    subgraph CORE["🔴 Tier 1: Always-On Core"]
        GW["API Gateway (05)
Traefik + Rate Limit"]
        AUTH["Auth Service (06) 🔐
JWT + OAuth2"]
        RISK["Risk Engine (11) 🔴
Real-Time Hypo/Hyper"]
        STREAM["Edge Streaming (19) ⚡
WebRTC Backbone"]
        CGM["CGM Sync (26) ⚡
Dexcom/Libre"]
    end

    subgraph BROKER["🔄 Event Bus (02)"]
        BUS["System Orchestrator
Kafka/EventBridge"]
    end

    subgraph ONDEMAND["🟡 Tier 2: On-Demand Serverless"]
        USERS["User Service (07)"]
        NOTIF["Notifications (08)"]
        NUTRITION["Nutrition Engine (09)"]
        ENERGY["Energy Engine (10)"]
        GLUC_IN["Glucose Input (12)"]
        WEAR["Wearable Sync (13)"]
        
        subgraph AI_VISION["AI Vision (GPU)"]
            FREC["Food Recog (14)"]
            FING["Ingredient Det (15)"]
            FPOR["Portion Est (16)"]
            FCOOK["Cooking State (17)"]
        end

        subgraph ASSISTANTS["AI Assistants"]
            VOICE["Voice (20)"]
            VISION["Vision (21)"]
            COOK_AST["Cooking (22)"]
        end

        CTX["Context Engine (23)"]
        PRED["Glucose Pred (24)"]
        REC["Diet Recommend (25)"]
        PERS["Personalization (27)"]
        POST["Postmeal Analysis (28)"]
    end

    subgraph BATCH["🔵 Tier 3: Async Batch"]
        TWIN["Digital Twin (29) 🔬
Neural ODE"]
        SIM["Simulation (30)"]
        FL["Fed Learning (31) 🔬"]
    end

    subgraph DATA["Data Layer"]
        PG[("PostgreSQL
Patient Records")]
        INFLUX[("InfluxDB
Time-Series")]
        REDIS[("Redis
Cache/Sessions")]
        MINIO[("MinIO
Images")]
        MLFLOW[("MLflow
Models")]
    end

    GLASSES -->|WebRTC| STREAM
    MOB <-->|REST/WS| GW
    WEB <-->|REST| GW
    STREAM -->|REST| GW

    GW --> AUTH
    GW --> BUS
    GW --> USERS
    GW --> FREC
    GW --> PRED
    GW --> REC
    GW --> VOICE
    GW --> VISION
    GW --> TWIN
    GW --> WEAR

    AUTH --> USERS
    
    FREC --> FING
    FING --> FPOR
    FREC --> NUTRITION
    FPOR --> NUTRITION
    
    WEAR --> ENERGY
    NUTRITION --> CTX
    ENERGY --> CTX
    GLUC_IN --> CTX
    
    CGM --> RISK
    CGM --> PRED
    CTX --> RISK
    CTX --> REC
    PRED --> REC
    PRED --> RISK
    
    RISK -->|Event| BUS
    BUS -.->|Wake up| NOTIF
    
    REC --> PERS
    REC --> POST
    REC --> TWIN
    
    FCOOK --> COOK_AST
    
    CORE <--> REDIS
    ONDEMAND <--> PG
    ONDEMAND <--> INFLUX
    AI_VISION <--> MINIO
    TWIN <--> REDIS
    
    FREC -.->|Gradients| FL
    PRED -.->|Gradients| FL
    FL -.->|Aggregate| MLFLOW
    MLFLOW -.->|Serve Model| FREC
    MLFLOW -.->|Serve Model| PRED
```

---

## 🔗 Service Dependency Graph

```mermaid
graph LR
    R03["03 mobile"]
    R04["04 dashboard"]
    R18["18 esp32-glasses"]
    R19["19 edge-streaming"]
    R05["05 api-gateway"]
    R06["06 auth"]
    R02["02 orchestrator (Event Bus)"]
    R07["07 users"]
    R08["08 notifications"]
    R09["09 nutrition"]
    R10["10 energy"]
    R11["11 risk 🔴"]
    R12["12 glucose-input"]
    R13["13 wearable-sync"]
    R14["14 food-rec"]
    R15["15 ingredient"]
    R16["16 portion"]
    R17["17 cooking-ai"]
    R20["20 voice"]
    R21["21 vision"]
    R22["22 cooking-assist"]
    R23["23 context"]
    R24["24 glucose-pred"]
    R25["25 diet-recommend"]
    R26["26 cgm ⚡"]
    R27["27 personalization"]
    R28["28 postmeal"]
    R29["29 digital-twin 🔬"]
    R30["30 simulation"]
    R31["31 federated 🔬"]

    R03 --> R05
    R04 --> R05
    R18 --> R19
    R19 --> R05
    
    R05 --> R06
    R05 --> R02
    
    R02 -.->|Events| R07
    R02 -.->|Events| R08
    R02 -.->|Events| R11
    R02 -.->|Events| R23
    
    R14 --> R09
    R15 --> R16
    R16 --> R09
    
    R13 --> R10
    R09 --> R23
    R10 --> R23
    R12 --> R23
    
    R23 --> R11
    R26 --> R11
    R26 --> R24
    
    R11 --> R08
    
    R24 --> R25
    R23 --> R25
    
    R14 -.-> R20
    R14 -.-> R21
    R17 -.-> R22
    
    R24 -.-> R29
    R25 -.-> R29
    R29 <--> R30
    
    R25 -.-> R27
    R25 -.-> R28
    
    R14 -.-> R31
    R24 -.-> R31
```

---

## 📈 Build Order & Complexity


| Priority | Repo | ⭐ Difficulty | Phase | Demo Value |
|---|---|---|---|---|
| 1 | `01` platform-architecture | ⭐ | Pre-Phase | — |
| 2 | `02` system-orchestrator | ⭐⭐ | Pre-Phase | — |
| 3 | `03` mobile-app | ⭐⭐ | Phase 1 | **Very High** |
| 4 | `04` web-dashboard | ⭐⭐ | Phase 1 | **Very High** |
| 5 | `05` api-gateway | ⭐⭐ | Phase 2 | Low |
| 6 | `06` auth-service | ⭐⭐ | Phase 2 | Low |
| 7 | `07` user-service | ⭐⭐ | Phase 2 | Low |
| 8 | `08` notification-service | ⭐⭐ | Phase 2 | Low |
| 9 | `09` nutrition-engine | ⭐⭐⭐ | Phase 3 | High |
| 10 | `10` energy-engine | ⭐⭐⭐ | Phase 3 | High |
| 11 | `11` risk-engine | ⭐⭐⭐⭐ | Phase 3 | High |
| 12 | `12` glucose-input-service | ⭐⭐ | Phase 3 | High |
| 13 | `13` wearable-sync | ⭐⭐⭐ | Phase 3 | High |
| 14 | `14` food-recognition | ⭐⭐⭐ | Phase 4 | **Very High** |
| 15 | `15` ingredient-detection | ⭐⭐⭐ | Phase 4 | High |
| 16 | `16` portion-estimation | ⭐⭐⭐ | Phase 4 | High |
| 17 | `17` cooking-state-ai | ⭐⭐⭐⭐ | Phase 4 | High |
| 18 | `18` esp32-glasses | ⭐⭐⭐ | Phase 5 | High |
| 19 | `19` edge-streaming | ⭐⭐⭐ | Phase 5 | High |
| 20 | `20` voice-assistant | ⭐⭐⭐ | Phase 5 | High |
| 21 | `21` vision-assistant | ⭐⭐⭐⭐ | Phase 5 | High |
| 22 | `22` cooking-assistant | ⭐⭐⭐⭐ | Phase 5 | High |
| 23 | `23` context-engine | ⭐⭐⭐⭐ | Phase 6 | High |
| 24 | `24` glucose-prediction | ⭐⭐⭐⭐ | Phase 6 | High |
| 25 | `25` diet-recommendation | ⭐⭐⭐⭐ | Phase 6 | **Very High** |
| 26 | `26` cgm-integration | ⭐⭐⭐ | Phase 6 | High |
| 27 | `27` personalization-engine | ⭐⭐⭐⭐ | Phase 7 | High |
| 28 | `28` postmeal-analysis | ⭐⭐⭐ | Phase 7 | High |
| 29 | `29` digital-twin | ⭐⭐⭐⭐⭐ | Phase 7 | High |
| 30 | `30` simulation-engine | ⭐⭐⭐⭐ | Phase 7 | High |
| 31 | `31` federated-learning | ⭐⭐⭐⭐⭐ | Phase 7 | High |
| 32 | `32` devops | ⭐⭐⭐⭐ | Phase 8 | Low |
| 33 | `33` observability | ⭐⭐⭐ | Phase 8 | Low |

---

## 🗺️ Development Phases

```mermaid
gantt
    title GlucoVision — 33 Repo Hybrid Build Roadmap
    dateFormat YYYY-MM
    section Pre-Phase — Foundation
    01 Platform Docs                  :done, 2026-05, 1M
    02 System Orchestrator            :2026-05, 1M

    section Phase 1 — Core UI
    03 Mobile App                     :2026-06, 2M
    04 Web Dashboard                  :2026-06, 1M

    section Phase 2 — API Foundation
    05 API Gateway                    :2026-06, 1M
    06 Auth Service                   :2026-06, 1M
    07 User Service                   :2026-07, 1M
    08 Notification Service           :2026-07, 1M

    section Phase 3 — Health Engine
    09 Nutrition Engine               :2026-08, 1M
    10 Energy Engine                  :2026-08, 1M
    11 Risk Engine                    :2026-09, 1M
    12 Glucose Input Service          :2026-09, 1M
    13 Wearable Sync                  :2026-09, 1M

    section Phase 4 — AI Vision
    14 Food Recognition               :2026-10, 2M
    15 Ingredient Detection           :2026-10, 1M
    16 Portion Estimation             :2026-11, 2M
    17 Cooking State AI               :2026-11, 1M

    section Phase 5 — Edge Hardware
    18 ESP32 Glasses                  :2026-12, 2M
    19 Edge Streaming                 :2026-12, 1M
    20 Voice Assistant                :2027-01, 1M
    21 Vision Assistant               :2027-01, 1M
    22 Cooking Assistant              :2027-01, 1M

    section Phase 6 — Intelligence
    23 Context Engine                 :2027-02, 1M
    24 Glucose Prediction             :2027-02, 2M
    25 Diet Recommendation            :2027-03, 1M
    26 CGM Integration                :2027-03, 1M

    section Phase 7 — Research
    27 Personalization Engine         :2027-04, 1M
    28 Postmeal Analysis              :2027-04, 1M
    29 Digital Twin                   :2027-05, 3M
    30 Simulation Engine              :2027-05, 2M
    31 Federated Learning             :2027-06, 3M

    section Phase 8 — Hardening
    32 DevOps K8s / Serverless        :2027-07, 2M
    33 Observability                  :2027-07, 1M
```
