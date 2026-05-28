# 📅 GlucoVision: 120-Day Feature Implementation Plan

Based on the 21-repository microservice architecture and the research proposal, this document breaks down the GlucoVision platform into an extended **120-day implementation plan**. This gives you more breathing room to implement complex microservices, CI/CD pipelines, and deep learning models at a realistic pace of **one feature per day**.

---

## 🏗️ Phase 1: Foundation & Infrastructure (Days 1–25)
*Focus: DevOps, secure networking, identity management, and core data storage.*

- [ ] **Day 1: Platform Scaffolding (Repo 21 - `devops-infra`)** - Initialize the core monorepo structure (or git submodules) for the 21 services.
- [ ] **Day 2: Kubernetes Setup (Repo 21 - `devops-infra`)** - Write Terraform scripts to provision a local/cloud K8s cluster.
- [ ] **Day 3: CI/CD Pipelines (Repo 21 - `devops-infra`)** - Setup GitHub Actions/GitLab CI for automated Docker builds.
- [ ] **Day 4: Secret Management (Repo 21 - `devops-infra`)** - Integrate HashiCorp Vault or similar for injecting secrets securely.
- [ ] **Day 5: Gateway Init (Repo 08 - `api-gateway`)** - Setup NGINX/Envoy base configuration for service routing.
- [ ] **Day 6: Gateway Rate Limiting (Repo 08 - `api-gateway`)** - Implement Redis-based IP rate limiting to prevent abuse.
- [ ] **Day 7: Auth Database (Repo 05 - `auth-service`)** - Initialize PostgreSQL database schema for secure credential storage.
- [ ] **Day 8: JWT Implementation (Repo 05 - `auth-service`)** - Build the JWT issuing and validation logic.
- [ ] **Day 9: Role-Based Access (Repo 05 - `auth-service`)** - Implement middleware to separate Patient, Clinician, and Admin scopes.
- [ ] **Day 10: OAuth Integration (Repo 05 - `auth-service`)** - Setup Google/Apple sign-in providers.
- [ ] **Day 11: User Profiles DB (Repo 06 - `user-service`)** - Create MongoDB schema for flexible user health profiles (age, weight, conditions).
- [ ] **Day 12: User CRUD API (Repo 06 - `user-service`)** - Implement REST endpoints for updating patient biometrics.
- [ ] **Day 13: Dietary Preferences (Repo 06 - `user-service`)** - Build logic to store and retrieve specific Sri Lankan dietary constraints.
- [ ] **Day 14: Notification Core (Repo 07 - `notification-service`)** - Setup RabbitMQ/Kafka listener for incoming alert events.
- [ ] **Day 15: Push Notifications (Repo 07 - `notification-service`)** - Integrate Firebase Cloud Messaging (FCM) for mobile push alerts.
- [ ] **Day 16: Email Dispatcher (Repo 07 - `notification-service`)** - Integrate SendGrid/AWS SES for weekly health summary emails.
- [ ] **Day 17: Web Dashboard Setup (Repo 04 - `web-dashboard`)** - Initialize React (Vite/Next.js) with Tailwind CSS/Material UI.
- [ ] **Day 18: Clinician Auth UI (Repo 04 - `web-dashboard`)** - Build the secure login portal for healthcare providers.
- [ ] **Day 19: Patient List View (Repo 04 - `web-dashboard`)** - Build the UI table to list registered patients and basic status.
- [ ] **Day 20: Data Collect Setup (Repo 02 - `data-collect-app`)** - Initialize cross-platform Flutter app specifically for food image gathering.
- [ ] **Day 21: Camera Module (Repo 02 - `data-collect-app`)** - Implement native camera controls to capture 2D images.
- [ ] **Day 22: Depth Mapping UI (Repo 02 - `data-collect-app`)** - Add iOS LiDAR / Android Depth API capture logic.
- [ ] **Day 23: Data Annotation Form (Repo 02 - `data-collect-app`)** - Build UI to tag images with actual food weights/labels.
- [ ] **Day 24: Secure Image Upload (Repo 02 - `data-collect-app`)** - Implement AWS S3 / Cloudinary upload pipelines.
- [ ] **Day 25: Dataset Management API (Repo 02 - `data-collect-app`)** - Build backend endpoints to organize collected dataset batches.

---

## 📱 Phase 2: Mobile App Foundation & Food AI (Days 26–55)
*Focus: Patient mobile experience and integrating the Sri Lankan food recognition pipeline.*

- [ ] **Day 26: Mobile Scaffold (Repo 03 - `glucovision-mobile`)** - Setup Flutter project, Riverpod, and folder architecture.
- [ ] **Day 27: Design System (Repo 03 - `glucovision-mobile`)** - Implement core typography, colors, and the Orbital Sync Loader.
- [ ] **Day 28: Mobile Auth Flow (Repo 03 - `glucovision-mobile`)** - Build Login, Signup, and JWT persistent storage.
- [ ] **Day 29: Patient Onboarding (Repo 03 - `glucovision-mobile`)** - Build screens for entering baseline health data (weight, diabetes type).
- [ ] **Day 30: Core Dashboard UI (Repo 03 - `glucovision-mobile`)** - Implement the main bottom navigation and home summary view.
- [ ] **Day 31: Food AI Setup (Repo 09 - `food-recognition`)** - Initialize Python FastAPI backend and load the pre-trained ResNet/ViT model.
- [ ] **Day 32: Image Preprocessing (Repo 09 - `food-recognition`)** - Implement tensor reshaping, normalization, and noise reduction on incoming images.
- [ ] **Day 33: Classification API (Repo 09 - `food-recognition`)** - Build the REST endpoint returning the Top-3 Sri Lankan food predictions.
- [ ] **Day 34: Nutrition Database Sync (Repo 09 - `food-recognition`)** - Connect the model output to a database of macros (Carbs, Protein, Fat, GI).
- [ ] **Day 35: Portion AI Setup (Repo 10 - `portion-estimation`)** - Initialize Python backend for 3D depth processing (Mask R-CNN).
- [ ] **Day 36: Depth Map Segmentation (Repo 10 - `portion-estimation`)** - Isolate food items from the plate background using depth data.
- [ ] **Day 37: Geometric Volume (Repo 10 - `portion-estimation`)** - Calculate cm³ volume of segmented food blobs.
- [ ] **Day 38: Calorie Calculation (Repo 10 - `portion-estimation`)** - Cross-reference volume with food density metrics to estimate mass/calories.
- [ ] **Day 39: Mobile Camera Integr. (Repo 03 - `glucovision-mobile`)** - Embed the device camera into the patient app for meal logging.
- [ ] **Day 40: Mobile Edge Detection (Repo 03 - `glucovision-mobile`)** - Add on-device UI guides to help patients center their food.
- [ ] **Day 41: Mobile Network Layer (Repo 03 - `glucovision-mobile`)** - Build Dio/HTTP interceptors for sending images to Food/Portion APIs.
- [ ] **Day 42: Meal Review UI (Repo 03 - `glucovision-mobile`)** - Build a screen showing recognized food and allowing manual corrections.
- [ ] **Day 43: Meal Logging API (Repo 06 - `user-service`)** - Implement endpoint to save the confirmed meal to the user's daily log.
- [ ] **Day 44: Daily Macro Chart (Repo 03 - `glucovision-mobile`)** - Build UI to visualize calories/carbs consumed vs. daily goals.
- [ ] **Day 45: Cooking AI Init (Repo 11 - `cooking-ai`)** - Setup FastAPI and 3D-CNN architecture for video processing.
- [ ] **Day 46: Video Stream Receiver (Repo 11 - `cooking-ai`)** - Implement WebRTC/WebSocket to ingest live camera frames.
- [ ] **Day 47: Cooking State Inference (Repo 11 - `cooking-ai`)** - Run inference to detect actions (e.g., adding sugar, boiling).
- [ ] **Day 48: State Aggregation (Repo 11 - `cooking-ai`)** - Compile sequential actions into a cooking summary.
- [ ] **Day 49: ESP32 Firmware (Repo 17 - `esp32-cam`)** - Write C++ code for ESP32-CAM to connect to WiFi and capture MJPEG.
- [ ] **Day 50: ESP32 Streaming (Repo 17 - `esp32-cam`)** - Implement secure WebSocket streaming from ESP32 to Cooking AI.
- [ ] **Day 51: ESP32 Power Optimization (Repo 17 - `esp32-cam`)** - Add deep sleep and motion-triggered wakeups.
- [ ] **Day 52: Cooking AI Sync (Repo 06 - `user-service`)** - Link inferred cooking macros to the user's meal log.
- [ ] **Day 53: Mobile Cooking Dashboard (Repo 03 - `glucovision-mobile`)** - Build a UI tab to view live ESP32 camera feeds.
- [ ] **Day 54: Recipe Breakdown UI (Repo 03 - `glucovision-mobile`)** - Show the patient the macros of the meal they are actively cooking.
- [ ] **Day 55: Phase 2 Integration Test (All)** - E2E test from ESP32 capture -> Cooking AI -> User Service -> Mobile App.

---

## 🩸 Phase 3: Health AI & Medical Systems (Days 56–85)
*Focus: Securely ingesting continuous glucose data, forecasting spikes, and triggering life-saving alerts.*

- [ ] **Day 56: CGM Webhook Security (Repo 14 - `cgm-integration`)** - Implement HMAC validation for incoming Dexcom/Libre webhooks.
- [ ] **Day 57: Dexcom OAuth Flow (Repo 14 - `cgm-integration`)** - Build the OAuth2 flow allowing users to link their CGM accounts.
- [ ] **Day 58: CGM Data Ingestion (Repo 14 - `cgm-integration`)** - Write parsers to standardize differing CGM JSON payloads into a unified format.
- [ ] **Day 59: Event Streaming Pipeline (Repo 14 - `cgm-integration`)** - Setup Apache Kafka / AWS Kinesis to broadcast live glucose readings.
- [ ] **Day 60: Prediction Engine Init (Repo 12 - `glucose-pred`)** - Setup Python microservice and load the Bi-LSTM / Transformer model.
- [ ] **Day 61: Time-Series Preprocessing (Repo 12 - `glucose-pred`)** - Implement algorithms to handle missing CGM data points and smooth noise.
- [ ] **Day 62: Continuous Forecasting (Repo 12 - `glucose-pred`)** - Implement logic to constantly consume Kafka topics and predict glucose 2-hours out.
- [ ] **Day 63: Prediction Output Stream (Repo 12 - `glucose-pred`)** - Publish the forecasted glucose curves back to a new Kafka topic.
- [ ] **Day 64: Risk Engine Init (Repo 13 - `risk-alerts`)** - Initialize Go/Rust microservice for high-speed, isolated medical rule evaluation.
- [ ] **Day 65: Hypoglycemia Rules (Repo 13 - `risk-alerts`)** - Write logic to detect rapid drops and predict imminent hypoglycemia.
- [ ] **Day 66: Hyperglycemia Rules (Repo 13 - `risk-alerts`)** - Write logic to detect sustained high glucose based on recent meal inputs.
- [ ] **Day 67: Alert Throttling (Repo 13 - `risk-alerts`)** - Implement state management (Redis) to prevent alert fatigue (no duplicate alerts within 30 mins).
- [ ] **Day 68: Alert Dispatch to Queue (Repo 13 - `risk-alerts`)** - Route validated critical alerts to the Notification Service payload.
- [ ] **Day 69: Mobile CGM Link UI (Repo 03 - `glucovision-mobile`)** - Build screens for users to connect Dexcom/Libre credentials.
- [ ] **Day 70: Mobile Charting Setup (Repo 03 - `glucovision-mobile`)** - Integrate FL Chart (or similar) to plot time-series data.
- [ ] **Day 71: Live Glucose Graph (Repo 03 - `glucovision-mobile`)** - Build the main dashboard graph showing the past 12 hours of CGM data.
- [ ] **Day 72: Predictive Graph Overlay (Repo 03 - `glucovision-mobile`)** - Overlay the forecasted 2-hour glucose curve (dotted line) on the main chart.
- [ ] **Day 73: Push Notification Handler (Repo 03 - `glucovision-mobile`)** - Implement foreground/background interception of FCM risk alerts.
- [ ] **Day 74: Critical Alert UI (Repo 03 - `glucovision-mobile`)** - Build full-screen, highly visible emergency warnings for severe hypoglycemia.
- [ ] **Day 75: Web Dashboard Patient Detail (Repo 04 - `web-dashboard`)** - Build clinician UI to view a specific patient's historical CGM data.
- [ ] **Day 76: Web Dashboard Live Monitoring (Repo 04 - `web-dashboard`)** - Implement WebSocket connection to show clinicians live patient glucose.
- [ ] **Day 77: Clinician Alert UI (Repo 04 - `web-dashboard`)** - Build a triage dashboard for clinicians to see which patients are at risk.
- [ ] **Day 78: Medical Export API (Repo 14 - `cgm-integration`)** - Build API to generate HIPAA-compliant PDF reports of glucose trends.
- [ ] **Day 79: Mobile Report Download (Repo 03 - `glucovision-mobile`)** - Add capability for patients to download and share their reports.
- [ ] **Day 80: Phase 3 Integration Test (All)** - E2E test from Mock CGM Webhook -> Kafka -> Predictor -> Risk Engine -> Mobile Push Alert.

---

## 🧠 Phase 4: Intelligence & Edge Computing (Days 81–100)
*Focus: Personalized AI recommendations, LLM chatbots, and wearable integrations.*

- [ ] **Day 81: Recommendation Engine Init (Repo 15 - `recommendation-engine`)** - Setup Python/Go service for generating actionable advice.
- [ ] **Day 82: Context Aggregation (Repo 15 - `recommendation-engine`)** - Build API to fetch recent meals, CGM data, and biometrics synchronously.
- [ ] **Day 83: Reinforcement Learning Logic (Repo 15 - `recommendation-engine`)** - Implement RL model to score dietary interventions based on glucose outcomes.
- [ ] **Day 84: Activity Recommendations (Repo 15 - `recommendation-engine`)** - Generate workout suggestions (e.g., "15 min walk to reduce spike") based on current glucose velocity.
- [ ] **Day 85: Alternative Meal API (Repo 15 - `recommendation-engine`)** - Suggest low-GI Sri Lankan alternatives to high-risk logged meals.
- [ ] **Day 86: Assistant Service Init (Repo 16 - `assistant-service`)** - Setup Python service integrating with LLaMA-3 (via local hosting or API).
- [ ] **Day 87: System Prompts (Repo 16 - `assistant-service`)** - Design and secure the LLM prompt to restrict advice strictly to diabetes management.
- [ ] **Day 88: Chat Session Memory (Repo 16 - `assistant-service`)** - Implement Redis/Postgres conversational memory so the LLM remembers previous messages.
- [ ] **Day 89: RAG Integration (Repo 16 - `assistant-service`)** - Implement Retrieval-Augmented Generation to inject the user's latest CGM/meal data into the LLM context.
- [ ] **Day 90: Chatbot API endpoints (Repo 16 - `assistant-service`)** - Finalize the WebSocket/REST API for real-time streaming LLM responses.
- [ ] **Day 91: Wearable Sync Init (Repo 18 - `wearable-sync`)** - Setup service to ingest steps, heart rate, and sleep data.
- [ ] **Day 92: Activity Data Normalization (Repo 18 - `wearable-sync`)** - Map data from Apple Health / Google Fit into a standard format.
- [ ] **Day 93: Wearable Kafka Producer (Repo 18 - `wearable-sync`)** - Broadcast activity events to update the Recommendation Engine context.
- [ ] **Day 94: Mobile BLE Sync (Repo 03 - `glucovision-mobile`)** - Implement Flutter BLE scanning to sync directly with generic smart bands.
- [ ] **Day 95: Apple Health/Health Connect (Repo 03 - `glucovision-mobile`)** - Integrate native OS health SDKs into the mobile app.
- [ ] **Day 96: Mobile Chatbot UI (Repo 03 - `glucovision-mobile`)** - Build a chat interface with typing indicators for the LLaMA-3 assistant.
- [ ] **Day 97: Mobile Suggestion Cards (Repo 03 - `glucovision-mobile`)** - Build UI cards displaying contextual recommendations (e.g., "Walk now").
- [ ] **Day 98: Mobile Activity Dashboard (Repo 03 - `glucovision-mobile`)** - Build graphs to visualize steps and heart rate alongside glucose.
- [ ] **Day 99: Mobile Meal Alternatives (Repo 03 - `glucovision-mobile`)** - UI to swap planned meals for the AI-suggested low-GI options.
- [ ] **Day 100: Phase 4 Integration Test (All)** - E2E test from Apple Health -> Recommendation Engine -> Chatbot -> Mobile UI.

---

## 🔬 Phase 5: Research & Final Polish (Days 101–120)
*Focus: Deep physiological simulations, federated privacy, and production readiness.*

- [ ] **Day 101: Digital Twin Init (Repo 19 - `digital-twin`)** - Setup PyTorch/JAX environment for Neural ODE modeling.
- [ ] **Day 102: Patient State Mapping (Repo 19 - `digital-twin`)** - Create pipelines mapping 30-day historical data into ODE initial parameters.
- [ ] **Day 103: Simulation Solver (Repo 19 - `digital-twin`)** - Implement the differential equation solver to project 24/48-hour physiological states.
- [ ] **Day 104: Scenario Intervention API (Repo 19 - `digital-twin`)** - Build API accepting "What-If" inputs (e.g., +50g carbs at 2PM) and returning new curves.
- [ ] **Day 105: Federated Server Setup (Repo 20 - `federated-learning`)** - Deploy the Flower framework aggregation server.
- [ ] **Day 106: Differential Privacy Layer (Repo 20 - `federated-learning`)** - Implement ε-DP noise addition during weight aggregation to protect patient privacy.
- [ ] **Day 107: Edge Model Export (Repo 20 - `federated-learning`)** - Optimize and convert the global prediction model to TFLite for mobile devices.
- [ ] **Day 108: Mobile On-Device Training (Repo 03 - `glucovision-mobile`)** - Implement Flutter background tasks to fine-tune the TFLite model on local data.
- [ ] **Day 109: Mobile Weight Upload (Repo 03 - `glucovision-mobile`)** - Securely upload locally trained model gradients to the Federated Server.
- [ ] **Day 110: Web Dashboard Twin UI (Repo 04 - `web-dashboard`)** - Build the "Scenario Simulator" UI for clinicians to test diets on digital twins.
- [ ] **Day 111: Web Dashboard Federated Stats (Repo 04 - `web-dashboard`)** - UI for researchers to view global model accuracy and training node status.
- [ ] **Day 112: Mobile "What-If" Sliders (Repo 03 - `glucovision-mobile`)** - Build interactive sliders for patients to preview meal impacts on their glucose.
- [ ] **Day 113: Load Testing Preparation (Repo 21 - `devops-infra`)** - Write JMeter/k6 scripts targeting the API Gateway and CGM ingestion pipelines.
- [ ] **Day 114: Security Auditing (All Repos)** - Run SAST tools and verify JWT/OAuth scopes across all 21 services.
- [ ] **Day 115: Medical Compliance Audit (Repo 12/13)** - Verify the strict isolation and fallback mechanisms of the Prediction and Risk engines.
- [ ] **Day 116: Bug Fix & UI Polish 1 (Mobile)** - Resolve Flutter UI jank, optimize orbital sync animations, fix navigation bugs.
- [ ] **Day 117: Bug Fix & UI Polish 2 (Web/Edge)** - Resolve React dashboard issues and ESP32 streaming latency.
- [ ] **Day 118: Production Infrastructure Sync (Repo 21 - `devops-infra`)** - Apply Terraform to provision the actual production/staging cloud environments.
- [ ] **Day 119: System Stress Test (All)** - Run massive simulated data ingestion representing 10,000 active patients.
- [ ] **Day 120: Go/No-Go Deployment Check (All)** - Final validation of all features and commencement of clinical trial data collection.

---

> [!IMPORTANT]
> **Daily Strategy:** Before writing code for any given day, consult the corresponding `repo-docs/XX-service-name.md` within `01-glucovision-platform-architecture` to ensure you align with the predetermined schemas and design patterns.
