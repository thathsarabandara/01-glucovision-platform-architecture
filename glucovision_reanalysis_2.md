# GlucoVision — 21-Repo Architecture Plan (v2: Autonomous Guardian)

> Based on: *Systematic Review of AI-Based PDAR System for Diabetic Individuals*  
> Strategy: **Shift from "Passive Tracker" to "Proactive Guardian Agent."**

---

## 🔍 The "Guardian Agent" Principle

The v2 architecture introduces the **Zero-Command Proactive Intervention** model. Instead of waiting for the user to ask questions, the system observes environmental and physiological data streams and intervenes autonomously.

| Feature | Passive Mode (v1) | Autonomous Guardian (v2) |
|---|---|---|
| **Interaction** | User-initiated (Request-Response) | System-initiated (Proactive Alert) |
| **Data Entry** | Manual/Camera capture click | **Zero-Click** (Live stream identification) |
| **Memory** | Session-based | **Adaptive Memory** (Handles "Remind me later") |
| **Simulation** | On-demand charts | **Auto-Simulation** (Digital Twin "What-If" on sight) |
| **Context** | User profile only | **Geo-Spatial Context** (Gym/Grocery mode) |

---

## 📦 All 21 Repositories (Refined)

```mermaid
graph TD
    subgraph EXISTING["Existing (4)"]
        R01["01 platform-architecture"]
        R02["02 data-collect-app"]
        R03["03 glucovision-mobile"]
        R04["04 web-dashboard"]
    end

    subgraph BACKEND["Backend Services (4)"]
        R05["05 auth-service 🔐"]
        R06["06 user-service"]
        R07["07 notification-service\nADAPTIVE — Snooze/Reschedule"]
        R08["08 api-gateway"]
    end

    subgraph FOOD_AI["Food AI Layer (3)"]
        R09["09 food-recognition-service\nAUTONOMOUS — Stream Inference"]
        R10["10 portion-estimation-service"]
        R11["11 cooking-ai-service\nPROACTIVE — Step Guidance"]
    end

    subgraph HEALTH_AI["Health AI Layer (3)"]
        R12["12 glucose-prediction-service 🔴"]
        R13["13 risk-alert-engine 🔴"]
        R14["14 cgm-integration-service ⚡"]
    end

    subgraph INTEL["Intelligence Layer (2)"]
        R15["15 recommendation-engine\nCONTEXT — Gym/Grocery GPS"]
        R16["16 assistant-service\nMASTER AGENT — LangGraph Orchestration"]
    end

    subgraph IOT["IoT & Edge (2)"]
        R17["17 esp32-cam-streaming\nMULTI-CAST — WebRTC to AI"]
        R18["18 wearable-sync-service"]
    end

    subgraph RESEARCH["Research-Grade (2)"]
        R19["19 digital-twin 🔬\nREAL-TIME — "What-If" Engine"]
        R20["20 federated-learning 🔬"]
    end

    subgraph INFRA["Infrastructure (1)"]
        R21["21 devops-infra"]
    end
```

---

## 🗂️ Refined Repo Details (New Capabilities)

### `03` glucovision-mobile *(Autonomous UI)*
- **Automatic Overlay**: Pops up Digital Twin projections and alerts without user interaction.
- **Guardian Dashboard**: Real-time visualization of proactive agent reasoning.
- **Geofencing**: Automatically triggers "Gym Mode" or "Grocery Mode" logic.

---

### `07` notification-service *(Adaptive)*
- **Conversational Rescheduling**: Allows `16` to reschedule notifications based on user excuses (e.g., *"remind me in 2 hours"*).
- **Intelligent Priority**: Escalates risk alerts while suppressing routine logs during exercise.

---

### `11` cooking-ai-service *(Proactive)*
- **Live Stream Analysis**: Analyzes ESP32-CAM video to detect "High Risk" cooking actions (e.g., adding too much salt/sugar) and triggers the Master Agent to intervene.

---

### `15` recommendation-engine *(Context-Aware)*
- **Gym Mode**: Adjusts carb-to-insulin advice based on live heart rate from `18` and GPS location.
- **Grocery Mode**: Provides a "Guardian Shopping List" when the user enters a supermarket.

---

### `16` assistant-service *(MASTER AGENT — The Brain)*
**Now the central orchestrator using LangChain/LangGraph.**
- **Adaptive Memory**: Remembers that the user's band is charging or they don't have a scale, avoiding repetitive nagging.
- **Proactive Intervention Manager**: Evaluates if the vision/health state warrants an interruption.
- **Digital Twin Interposer**: Automatically requests "What-If" simulations from `19` when new food is seen.

---

### `19` digital-twin *(Real-Time Engine)*
- **What-If API**: Low-latency endpoint to simulate the effect of a detected meal *before* ingestion.
- **Comparison Engine**: Shows "Detected Meal" vs "Recommended Alternative" projection curves.

---

## 🏗️ v2 System Architecture: The Proactive Loop

```mermaid
graph TB
    subgraph SENSORS["Guardian Sensors"]
        CAM["ESP32-CAM (17)\nLive Stream"]
        CGM["CGM (14)\nLive Glucose"]
        GPS["Mobile GPS (03)\nLocation Context"]
    end

    subgraph BRAIN["Master Agent (16)"]
        ORCH["LangGraph Orchestrator"]
        MEM["Adaptive Memory\n(Deferred Tasks)"]
        REASON["Intervention Reasoning"]
    end

    subgraph AGENTS["Sub-Agents"]
        FOOD["Food Agent (09/10)"]
        HEALTH["Health Agent (12/13)"]
        TWIN["Twin Agent (19)"]
        DIET["Context Agent (15)"]
    end

    CAM -->|Stream| FOOD
    CGM -->|Levels| HEALTH
    GPS -->|Location| DIET

    FOOD -->|Identify Food| ORCH
    HEALTH -->|Predict Spike| ORCH
    DIET -->|Gym/Grocery| ORCH

    ORCH -->|Request "What-If"| TWIN
    TWIN -->|Projection| ORCH

    ORCH -->|Proactive Voice| TTS["Voice: 'Wait, don't eat that yet!'"]
    ORCH -->|Auto-UI| UI["Pop-up Digital Twin Chart"]
    ORCH -->|Deferred| MEM["'Ask again in 2 hours'"]
```

---

## 🗺️ Build Order Updates (v2 Priority)

| Priority | Repo | New v2 Scope |
|---|---|---|
| **High** | `16` assistant-service | Implement LangGraph orchestrator + Memory modules. |
| **High** | `19` digital-twin | Optimize simulation latency for real-time "What-If" calls. |
| **Medium** | `03` mobile-app | Implement "Automatic Overlay" and "Geofencing" triggers. |
| **Medium** | `15` recommendation | Add situational logic for Gym/Grocery contexts. |

---

> [!IMPORTANT]
> **Difference from v1**: While v1 focused on building the *services*, v2 focuses on the **Intelligence Fusion**. The system no longer waits for a "Food Photo" button click—it analyzes the WebRTC stream in the background and speaks when it detects a risk.

> [!TIP]
> **Zero-Command UX**: The goal is for the user to interact with the system like a real human nutritionist who is "looking over their shoulder" during cooking or eating.
