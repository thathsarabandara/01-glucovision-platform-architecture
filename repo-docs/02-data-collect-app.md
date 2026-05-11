# 📸 Repo 02 — `data-collect-app`

> **Status:** Existing · Active  
> **Flag:** —  
> **Difficulty:** ⭐⭐  
> **Build Priority:** 2 (Foundation — demo value: Very High)

---

## Purpose

This repository is the **Sri Lankan food image dataset collection platform**. It drives the custom dataset that the food recognition service (09) will be fine-tuned on. Without high-quality, culturally-relevant labeled images, the AI pipeline cannot serve Sri Lankan patients accurately. This is the data flywheel that fuels the entire Food AI layer.

---

## Core Functionality

| Module | What It Does |
|---|---|
| Flutter Capture App | Mobile UI for capturing food images at multiple angles (0°, 30°, 60°, 90°) |
| Camera Controller | Enforces capture parameters: angle, plate type, spoon type, background type |
| FastAPI Backend | Receives image uploads, validates metadata, stores to filesystem |
| Record Service | Manages food record CRUD, nutrition metadata, normalized weight calculations |
| Image Renaming Engine | Non-duplicating deterministic filename generation based on food parameters |
| Gallery View | Grouped image browser by food entity with aggregate image counts |
| Bulk Import | Excel-based bulk food metadata import with type normalization |
| Admin Web Dashboard | Vite/React frontend for managing food records and image collections |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Food images | Flutter mobile camera | JPEG/PNG |
| Capture metadata | App user entry | JSON (angle, weight, plate, spoon, background) |
| Food nutrition data | Admin entry / Excel bulk import | JSON / XLSX |
| Food name / category | Admin entry | String |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Labeled food images | `09-food-recognition-service` (training data) | JPEG files + JSON metadata |
| Normalized nutrition records | `15-recommendation-engine` (via DB) | PostgreSQL / JSON |
| Food master list | `09`, `15` | REST API |
| Dataset statistics | `04-web-dashboard` | REST API |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| PostgreSQL | Persistent storage of food records and metadata |
| MinIO / Local filesystem | Image file storage |
| FastAPI | Backend REST API framework |
| React/Vite | Admin web dashboard frontend |
| Docker Compose | Local dev orchestration |
| IndexedDB (browser) | Client-side cache for offline sync |

---

## Communication

| Protocol | Used For |
|---|---|
| REST (HTTP/JSON) | Mobile app → backend API, web dashboard → backend API |
| Multipart/form-data | Image upload from Flutter to FastAPI |
| WebSocket | (Optional) real-time upload progress |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile Frontend | Flutter (Dart) |
| Web Frontend | React + Vite (TypeScript) |
| Backend API | FastAPI (Python) |
| Database | PostgreSQL |
| File Storage | Local filesystem / MinIO |
| Containerization | Docker + Docker Compose |
| State (browser) | IndexedDB (client cache) |

---

## AI/ML Knowledge

> No live inference — this repo **creates** the training data that AI models consume.

| Concern | Detail |
|---|---|
| Dataset quality | Controlled capture conditions (angle, background, lighting standardization) |
| Label schema | Food name, cuisine category, nutrition per 100g, capture parameters |
| Data balance | Multiple angles per food item ensures dataset diversity |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| PostgreSQL | Food records, metadata, nutrition data | Relational (normalized) |
| Filesystem / MinIO | Raw image files | JPEG/PNG with structured filenames |
| IndexedDB | Browser-side sync cache | Key-Value |

**Key data schema concepts:**
- Nutritional values stored per 100g → normalized to actual weight on record creation
- Image filenames encode: `{food_id}_{angle}_{weight}g_{plate}_{spoon}_{background}.jpg`

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Deployment | Docker Compose (dev); can be containerized for cloud |
| Storage volume | Grows with dataset size — plan MinIO early |
| Scalability | Single-instance backend sufficient for data collection phase |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Auth | Admin-only access (no patient data here) |
| Image access | Filesystem ACL or MinIO bucket policy |
| API security | Basic token auth or session for admin dashboard |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Upload throughput | Bulk image uploads from multiple field capture sessions |
| Image resizing | Resize/compress on upload to avoid storage bloat |
| IndexedDB sync | Conflict resolution for offline captures syncing to backend |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Unit Tests | Record service nutrition calculation logic |
| Integration Tests | FastAPI endpoint tests (upload, CRUD) |
| E2E | Flutter capture → backend stored → gallery visible |
| Data Validation | Bulk import type checking, nutrition range validation |

---

## Observability

| Concern | Detail |
|---|---|
| Logging | FastAPI structured logs (upload success/failure, image filename) |
| Metrics | Upload count, storage utilization, records per food type |
| Alerts | Storage capacity warning |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟡 Intermediate | Multi-platform (mobile + web + backend) coordination; dataset engineering decisions have downstream AI impact |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Full-stack multi-platform development (Flutter + React + FastAPI) |
| Dataset engineering for AI model training |
| Controlled data capture pipeline design |
| Nutritional data normalization logic |
| Docker Compose multi-service orchestration |
| Offline-first sync architecture (IndexedDB) |
