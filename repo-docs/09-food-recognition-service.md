# 🍽️ Repo 09 — `food-recognition-service`

> **Status:** Planned  
> **Flag:** GPU — Classification  
> **Difficulty:** ⭐⭐⭐  
> **Build Priority:** 9 (Phase 2 — Food AI)

---

## Purpose

The **computer vision engine that identifies what food is in a photo**. This service answers the question: *"What dish is this?"* It maps a patient's food photo to a food ID, which then unlocks the nutrition lookup needed for glucose impact prediction and personalized recommendations. It is separated from portion estimation (`10`) because 2D classification (what is it?) and 3D geometry (how much of it?) are fundamentally different model architectures, datasets, and GPU profiles.

**Research basis:** ResNet-50 (91% accuracy) and ViT-B16 (93% accuracy) from the systematic review [4][34], with fine-tuning on a custom Sri Lankan food dataset collected by `02`.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `classifier/` | ResNet-50 / ViT-B16 inference — returns food class + confidence |
| `sinhala_finetune/` | Domain adaptation layer fine-tuned on Sri Lankan food dataset from `02` |
| `nutrition_lookup/` | Maps predicted food ID → macronutrients (carbs, protein, fat, GI index) |
| `multi_item/` | Detects multiple food items in a single plate image |
| `preprocessing/` | Image normalization, augmentation, resize pipeline |
| `model_server/` | ONNX Runtime model serving endpoint |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Food image | Mobile app (03) via gateway (08) | JPEG/PNG (multipart) |
| Image URL | Alternative to upload | HTTPS URL |
| Top-K request param | Client | Integer (default: 3) |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Food class + confidence | Mobile app (03), `15` recommendation | JSON `{ food_id, name, confidence, top_k }` |
| Nutrition data | `15` recommendation, `12` glucose prediction | JSON `{ carbs_g, protein_g, fat_g, gi_index, calories }` |
| Bounding boxes | `10` portion-estimation (multi-item detection) | JSON array of bbox coordinates |
| Model metadata | `04` web dashboard (monitoring) | JSON `{ model_version, accuracy, inference_time }` |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `02` data-collect-app | Training data source (Sri Lankan food images) |
| `08` api-gateway | Request routing + auth forwarding |
| MinIO | Model weights storage + inference image cache |
| MLflow | Model versioning, experiment tracking, deployment registry |
| `19` federated-learning | Receives aggregated model updates (dashed dependency) |
| PostgreSQL / Nutrition5k DB | Food ID → nutrition lookup |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Image upload, inference request/response |
| Internal HTTP | Called by `15`, `10`, `12` for food recognition |
| MLflow API | Model registry fetch on startup |
| MinIO SDK | Model weight download |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Deep Learning | PyTorch 2.x |
| Models | ResNet-50, ViT-B16 (torchvision / HuggingFace Transformers) |
| Inference Optimization | ONNX Runtime (exported PyTorch → ONNX) |
| Image Processing | Pillow, torchvision.transforms |
| Model Registry | MLflow |
| Storage | MinIO (model weights + uploaded images) |
| GPU | NVIDIA CUDA (training + inference) |
| Containerization | Docker + NVIDIA Container Toolkit |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **Base model** | ResNet-50 pretrained on ImageNet → fine-tune on Food-101 → fine-tune on Sri Lankan dataset |
| **Alternative model** | ViT-B16 (Vision Transformer) — 93% accuracy per paper [34] |
| **Ensemble** | Combine ResNet-50 + ViT predictions for higher confidence |
| **Sri Lankan fine-tune** | Transfer learning on `02` dataset: rice dishes, curries, kottu, hoppers, etc. |
| **Multi-item detection** | YOLOv8 for detecting multiple dishes in one frame before classifying each |
| **Model drift** | Monitor accuracy drop over time using Prometheus + Grafana alerts |
| **Training pipeline** | PyTorch Lightning + MLflow experiment tracking |
| **FL integration** | Receives FedAvg-aggregated gradients from `19` to improve without centralizing data |

**Paper reference:**
- [4] GoogLeNet/ResNet achieves 91% top-1 accuracy on food classification
- [34] ViT-B16 achieves 93.4% on Food-101

---

## Data Knowledge

| Dataset | Purpose | Format |
|---|---|---|
| Food-101 | Pretraining base | 101,000 images, 101 classes |
| Custom Sri Lankan dataset (from `02`) | Domain fine-tuning | ~50,000 images, ~200+ classes |
| Nutrition5k | Nutrition label lookup table | CSV / PostgreSQL |
| USDA FoodData | Fallback nutrition database | API / CSV |

**Nutrition schema:**
```json
{
  "food_id": "lk_rice_curry_001",
  "name_en": "Rice and Curry",
  "name_si": "බතු සහ ව්‍යංජන",
  "carbs_per_100g": 28.5,
  "protein_per_100g": 5.2,
  "fat_per_100g": 3.1,
  "gi_index": 64,
  "calories_per_100g": 165
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| GPU requirement | NVIDIA GPU for inference (CUDA 12+); ONNX Runtime for CPU fallback |
| Model loading | Load ONNX model at startup, keep in memory |
| Inference latency | Target < 500ms per image (ONNX on GPU) |
| Scaling | Scale inference pods independently (GPU autoscaling in K8s) |
| Model updates | Blue-green deployment via MLflow model registry |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Auth | JWT validation via gateway (08) |
| Image validation | Reject non-image files, limit size to 10MB |
| Model integrity | Verify model checksum on download from MLflow |
| No PII in images | Ensure uploaded food images don't contain faces (privacy) |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| ONNX acceleration | PyTorch → ONNX export reduces inference latency 30–50% |
| Batch inference | Support batched requests for multiple images |
| Image preprocessing | GPU-accelerated preprocessing (torchvision CUDA pipeline) |
| Caching | Cache nutrition lookup by food_id (Redis, 1hr TTL) |
| p99 target | < 500ms end-to-end for single image inference |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Model Accuracy Tests | Evaluate on hold-out Sri Lankan food test set (target > 90%) |
| API Tests | FastAPI endpoint tests (upload → classification response) |
| Latency Tests | Locust load test — 50 concurrent image submissions |
| Regression Tests | Ensure new model version ≥ previous accuracy before promoting |
| Data Validation | Reject corrupt images, wrong MIME types |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Inference latency, confidence score distribution, top-1 class distribution |
| Model drift | MAPE on confidence scores; accuracy on labeled feedback from patients |
| Alerts | Accuracy drop > 5% → retrain trigger |
| Logging | Every prediction: food_id, confidence, inference_time, model_version |
| Tracing | OpenTelemetry span from gateway request → ONNX inference → response |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🟠 Advanced | Deep learning model training, transfer learning, ONNX optimization, Sri Lankan food domain expertise, federated learning integration |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| ResNet-50 / ViT transfer learning for food classification |
| Custom domain fine-tuning for Sri Lankan cuisine |
| ONNX Runtime deployment optimization |
| MLflow model registry and versioning |
| Multi-item food detection (YOLO) |
| Nutrition database integration |
| Federated learning model consumption |
| GPU-accelerated inference service |
