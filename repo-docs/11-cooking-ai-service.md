# 🍳 Repo 11 — `cooking-ai-service`

> **Status:** Planned  
> **Flag:** GPU — Temporal Vision  
> **Difficulty:** ⭐⭐⭐⭐  
> **Build Priority:** 15 (Phase 4 — Intelligence)

---

## Purpose

The **real-time cooking intelligence service** that watches a patient cook and provides step-by-step guidance with nutrition awareness. Uses 3D-CNN to understand cooking state from video (frying vs boiling vs steaming matters for nutrition outcomes). Separated from food recognition (`09`) because video/temporal models differ architecturally from static image classifiers — they process continuous streams, not single frames.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `cooking_state/` | 3D-CNN classifies cooking method from video stream (frying / boiling / steaming / raw) |
| `step_detector/` | Detects recipe step completion from video progression |
| `cooking_guidance/` | Real-time cooking instructions API via WebSocket (step-by-step) |
| `nutrition_adjustment/` | Adjusts nutrition estimates based on detected cooking method (frying adds fat) |
| `recipe_engine/` | Links cooking session to a recipe template with Sri Lankan dishes |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Video stream (cooking session) | Mobile camera (03) | MJPEG / WebRTC frames |
| Recipe selection | Patient via mobile (03) | Recipe ID |
| Food items (pre-cooking) | `09` food-recognition | JSON food_id list |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Current cooking state | Mobile app (03) via WebSocket | JSON `{ state: "frying", confidence: 0.92 }` |
| Step completion event | Mobile app (03) via WebSocket | JSON `{ step: 3, completed: true }` |
| Cooking guidance text | Mobile app (03), `16` assistant | JSON + WebSocket |
| Adjusted nutrition data | `15` recommendation, `12` glucose prediction | JSON (post-cook macros) |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `09` food-recognition | Pre-cooking food identification |
| `16` assistant-service | Narrates cooking guidance via voice |
| `08` api-gateway | WebSocket proxying |
| MinIO | 3D-CNN model weights |
| MLflow | Model versioning |

---

## Communication

| Protocol | Used For |
|---|---|
| WebSocket | Real-time cooking state stream to mobile |
| REST / HTTPS | Recipe lookup, session start/stop |
| Internal HTTP | Called by `16` assistant for cooking context |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) + WebSocket |
| Temporal Model | 3D-CNN (SlowFast / Video Swin Transformer) |
| Video Processing | OpenCV, PyTorch video |
| Deep Learning | PyTorch 2.x |
| Storage | MinIO (model weights) |
| GPU | NVIDIA CUDA |
| Containerization | Docker + NVIDIA Container Toolkit |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **3D-CNN** | SlowFast Networks or Video Swin Transformer — processes temporal video sequences |
| **Cooking state classes** | raw, frying, boiling, steaming, baking, completed |
| **Sri Lankan adaptation** | Fine-tune on Sri Lankan cooking video dataset (hoppers, kottu, curry preparation) |
| **Nutrition adjustment** | Frying: +15–25% fat; boiling: −10% water-soluble vitamins; steaming: minimal loss |
| **Step detection** | Temporal action segmentation — identify discrete recipe steps from continuous video |

---

## Data Knowledge

| Dataset | Purpose | Format |
|---|---|---|
| Kinetics-400 | 3D-CNN pretraining (action recognition) | MP4 video clips |
| Custom cooking video dataset | Fine-tuning on Sri Lankan cooking methods | MP4 + labels |
| Sri Lankan recipe database | Step templates | JSON |

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| GPU | Required — 3D-CNN inference is computationally heavy |
| WebSocket scaling | Sticky sessions for WebSocket cooking sessions |
| Stream processing | Frame sampling (every 2nd frame) to reduce GPU load |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Video privacy | Cooking session video never stored (process in-memory only) |
| Auth | JWT validation |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Frame sampling | 8–15 FPS input, process every 3rd frame for state detection |
| WebSocket latency | State update < 500ms from frame capture |
| Model size | Video Swin-T ~28M params — balance accuracy vs speed |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Model Tests | Cooking state accuracy > 88% on test video set |
| WebSocket Tests | Stable connection during 30-minute cooking session |
| Nutrition Adjustment Tests | Frying detection → fat adjustment within 5% of lab measurement |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | State detection latency, confidence per frame, session duration |
| Logging | Cooking session start/end, state transitions |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🔴 Advanced | Temporal video understanding + real-time WebSocket streaming + nutrition physics modeling |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| 3D-CNN / Video Swin Transformer for action recognition |
| Real-time video stream processing |
| WebSocket-based continuous inference |
| Cooking nutrition adjustment modeling |
| Sri Lankan recipe AI domain knowledge |
