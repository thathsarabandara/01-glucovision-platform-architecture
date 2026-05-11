# 📐 Repo 10 — `portion-estimation-service`

> **Status:** Planned  
> **Flag:** GPU — Geometry + Depth  
> **Difficulty:** ⭐⭐⭐  
> **Build Priority:** 12 (after food-recognition 09)

---

## Purpose

The **3D geometry engine that estimates how much food is in a photo**. This service answers: *"How many grams of this dish?"* It takes the food bounding boxes from `09` and uses depth sensing + volume calculation to estimate portion weight, which is essential for accurate nutrition calculation and glucose impact prediction. Separated from food recognition because RGB-D depth inference and 3D reconstruction are a completely different engineering domain from 2D classification.

**Research basis:** Mask R-CNN segmentation (94.1% IoU) [5], RGB-D depth estimation [11][12], volume-to-weight mapping [12][13], AR guidance [14][16].

---

## Core Functionality

| Module | What It Does |
|---|---|
| `ingredient_detection/` | Mask R-CNN semantic segmentation — pixel-level food boundary detection |
| `depth_estimation/` | RGB-D or monocular depth inference from single camera image |
| `volume_calculator/` | 3D point cloud → food volume (cm³) → weight (g) conversion |
| `ar_guidance/` | Compute AR overlay data (bounding box + size annotation) for mobile app |
| `portion_api/` | REST endpoint: image → grams per detected food item |
| `calibration/` | Reference object (plate, spoon) size calibration for scale estimation |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Food image | Mobile app (03) via gateway (08) | JPEG/PNG (multipart) |
| Bounding boxes from `09` | `09` food-recognition-service | JSON array of bbox + food_id |
| Depth image (if RGB-D device) | Smart glasses / ESP32-CAM (via `17`) | 16-bit depth PNG |
| Plate/reference object type | Capture metadata (from `02`) | String (plate_type) |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Portion weight per food item | `15` recommendation engine, `12` glucose prediction | JSON `{ food_id, weight_g }` |
| Segmentation mask | AR overlay in mobile app (03) | JSON polygon / PNG mask |
| AR overlay data | Mobile app (03) for ARCore/ARKit rendering | JSON `{ bbox, label, weight_g, confidence }` |
| 3D point cloud (debug) | `04` web dashboard (optional) | PLY / JSON |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `09` food-recognition | Food bounding boxes + food_id for each detected item |
| `08` api-gateway | Request routing + auth forwarding |
| MinIO | Model weights (Mask R-CNN, depth model) + intermediate image cache |
| MLflow | Model versioning |
| Open3D | 3D point cloud processing library |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Portion estimation request/response |
| Internal HTTP | Called by `15`, `12` post-food-recognition |
| WebRTC (via `17`) | Optional real-time stream from ESP32 glasses for live portion sizing |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Segmentation Model | Mask R-CNN (Detectron2 / torchvision) |
| Depth Estimation | MiDaS (monocular depth) / DepthAnything v2 |
| 3D Processing | Open3D |
| Image Processing | OpenCV, Pillow |
| Deep Learning | PyTorch 2.x |
| Model Registry | MLflow |
| Storage | MinIO |
| GPU | NVIDIA CUDA |
| Containerization | Docker + NVIDIA Container Toolkit |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **Mask R-CNN** | Instance segmentation — separate each food item at pixel level; 94.1% IoU per paper [5] |
| **Monocular depth** | MiDaS / DepthAnything v2 estimates depth from single RGB image (no special hardware) |
| **RGB-D depth** | If depth camera available (Intel RealSense / ESP32 with ToF), use measured depth for higher accuracy |
| **Volume calculation** | Integrate depth map over segmented food region → volume (cm³) |
| **Weight conversion** | Volume × density lookup per food type → weight (g); density table per food class |
| **AR overlay** | Return 3D bounding box + portion annotation for ARCore/ARKit rendering on mobile |
| **Reference calibration** | Use detected plate diameter (known size) as scale reference to calibrate pixel → cm |

**Paper reference:**
- [5] Mask R-CNN segmentation: 94.1% IoU
- [12] RGB-D depth-based volume estimation
- [13] Density-based weight conversion

---

## Data Knowledge

| Dataset | Purpose | Format |
|---|---|---|
| ECOC-based food segmentation dataset | Mask R-CNN training | Images + polygon masks |
| Custom Sri Lankan food depth dataset | Fine-tuning depth + volume estimates | RGB + depth image pairs |
| Food density table | Volume → weight conversion | CSV / PostgreSQL |

**Density lookup example:**
```json
{
  "food_id": "lk_rice_curry_001",
  "density_g_per_cm3": 0.85,
  "density_source": "empirical_measurement"
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| GPU requirement | NVIDIA GPU for Mask R-CNN inference |
| Inference latency | Target < 1000ms (segmentation + depth + volume pipeline) |
| Depth camera support | Optional: Intel RealSense SDK integration for lab setting |
| Scaling | K8s GPU node pool for inference pods |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Auth | JWT validation via gateway |
| Image size limit | 15MB max (depth images can be larger) |
| No face capture | Validate images don't contain faces |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Pipeline latency | Segmentation (400ms) + depth (200ms) + volume (100ms) = target < 1s |
| Parallel processing | Run Mask R-CNN and MiDaS depth estimation in parallel |
| ONNX optimization | Export Mask R-CNN and depth model to ONNX for faster inference |
| Caching | Cache portion results for same image hash (avoid recompute) |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Model Tests | Mask R-CNN IoU > 90% on test set |
| Volume Accuracy Tests | Volume error < 15% vs ground truth weight |
| API Tests | Image → portion weight JSON response |
| Edge Cases | Obscured food, mixed plate, no food detected |
| Calibration Tests | Plate reference calibration accuracy |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Inference latency per stage, confidence score, IoU on spot checks |
| Logging | Per-request: food items detected, weights estimated, model version |
| Alerts | IoU drop > 10% → model drift alert |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🔴 Advanced | Combination of instance segmentation, monocular depth estimation, 3D reconstruction, and AR data generation — frontier computer vision research |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Mask R-CNN instance segmentation implementation |
| Monocular depth estimation (MiDaS / DepthAnything) |
| 3D point cloud processing with Open3D |
| Volume-to-weight food physics modeling |
| AR overlay data generation for mobile ARCore/ARKit |
| RGB-D camera integration |
| Multi-stage inference pipeline optimization |
