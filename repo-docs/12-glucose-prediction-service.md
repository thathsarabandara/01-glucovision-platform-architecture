# 🩸 Repo 12 — `glucose-prediction-service`

> **Status:** Planned  
> **Flag:** 🔴 SEPARATE — Medically Critical  
> **Difficulty:** ⭐⭐⭐⭐  
> **Build Priority:** 13 (after CGM integration 14)

---

## Purpose

The **clinical blood glucose forecasting engine** — the most medically critical AI component in GlucoVision. Predicts a patient's blood glucose 30–120 minutes ahead so they can preemptively adjust meals and activity. A prediction error can cause a patient to fail to treat hypoglycemia or over-medicate hyperglycemia. For this reason, this service must have:
- **Independent rollback** (separate from all other services)
- **Own model versioning lifecycle** with drift monitoring
- **Separate scaling** (high-volume CGM polling)
- **Independent uptime SLA**

**Research basis:** Bi-LSTM [23][24], Transformer for time-series [25][26], LSTM + Transformer ensemble [24], sliding window preprocessing [27].

---

## Core Functionality

| Module | What It Does |
|---|---|
| `lstm_model/` | Bidirectional LSTM glucose forecasting (30/60/120 min horizons) |
| `transformer_model/` | Time-series Transformer for longer-horizon prediction |
| `ensemble/` | LSTM + Transformer weighted ensemble for robust predictions |
| `preprocessing/` | Sliding window creation, missing value imputation, normalization |
| `drift_monitor/` | MAPE-based model drift detection with alert triggers |
| `model_server/` | FastAPI inference endpoint with MLflow model loading |
| `feedback_loop/` | Capture actual vs predicted values to trigger retraining |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| CGM glucose readings (time-series) | `14` cgm-integration-service | InfluxDB time-series / JSON |
| Manual glucose entry | Mobile app (03) | JSON `{ value_mmol, timestamp }` |
| Meal data (food + portion) | `09` + `10` via `15` | JSON `{ carbs_g, gi_index, timestamp }` |
| Activity data | `17` edge-iot-service (wearable) | JSON `{ steps, heart_rate, timestamp }` |
| Patient health metadata | `06` user-service | JSON `{ diabetes_type, hba1c_target, medications }` |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Glucose forecast (30/60/120 min) | Mobile app (03), Web dashboard (04) | JSON `{ horizon_min, predicted_mmol, confidence_interval }` |
| Forecast time-series | `13` risk-alert-engine | JSON array (curve) |
| Forecast curve | `18` digital-twin | JSON + InfluxDB write |
| Drift metrics | `04` web dashboard, `08` monitoring | Prometheus metrics |
| Prediction vs actual | `19` federated-learning (gradient signal) | JSON (model update trigger) |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `14` cgm-integration | Primary glucose data source |
| `06` user-service | Patient health context for model conditioning |
| InfluxDB | Time-series glucose data store |
| MLflow | Model registry, experiment tracking, version promotion |
| `08` api-gateway | Request routing |
| `13` risk-alert-engine | Consumes forecast to trigger alerts |
| `18` digital-twin | Consumes forecast for simulation |
| `19` federated-learning | Consumes gradients from this service; sends back aggregated model |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Forecast request/response, model management API |
| InfluxDB client | Time-series glucose data read |
| Prometheus push | Drift metrics, prediction error rates |
| Internal HTTP | Called by `13`, `18`, `15` |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Deep Learning | PyTorch 2.x |
| LSTM Model | Custom Bi-LSTM (PyTorch nn.LSTM) |
| Transformer Model | Temporal Fusion Transformer (pytorch-forecasting) |
| Time-Series DB | InfluxDB |
| Model Registry | MLflow |
| Experiment Tracking | MLflow + TensorBoard |
| Preprocessing | NumPy, pandas |
| GPU | NVIDIA CUDA (training); CPU sufficient for inference |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **Bi-LSTM** | Bidirectional LSTM processes glucose time-series — captures both forward and backward temporal patterns |
| **Temporal Fusion Transformer** | Attention-based model excels at multi-horizon glucose prediction with interpretable attention weights |
| **Ensemble** | Weighted combination of LSTM + Transformer reduces variance; weights tuned per patient via personalization |
| **Sliding window** | Input: 2-hour rolling window of CGM readings (every 5 min = 24 data points) → predict next 30/60/120 min |
| **Imputation** | Missing CGM values interpolated using Kalman filter or forward-fill |
| **Model drift** | MAPE (Mean Absolute Percentage Error) computed daily; > 15% triggers retraining alert |
| **Personalization** | Per-patient model fine-tuning using `19` FL — global model + local adaptation |
| **Prediction horizons** | 30-min, 60-min, 120-min forecasts simultaneously |

**Paper references:**
- [23][24] Bi-LSTM for glucose prediction — RMSE < 20 mg/dL at 30-min horizon
- [25][26] Transformer for multi-step time-series forecasting
- [27] Sliding window preprocessing protocol

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| InfluxDB | Glucose time-series (CGM readings, manual entries) | `measurement=glucose, user_id=tag, value=field, time=timestamp` |
| MLflow | Model versions, training runs, metrics | Binary + JSON |
| PostgreSQL | Prediction logs, actual vs predicted | Relational |

**InfluxDB schema:**
```
measurement: glucose_readings
tags: user_id, source (cgm_dexcom / cgm_libre / manual)
fields: value_mmol (float), raw_mg_dl (float)
timestamp: nanosecond precision
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| GPU | Required for training; inference can run CPU (ONNX export) |
| Independent deployment | Must deploy, rollback, and scale independently from all other services |
| Model promotion | MLflow: staging → production with clinical validation gate |
| Retraining pipeline | Automated trigger on drift alert → MLflow training job → A/B shadow test → promote |
| Scaling | High CGM polling volume (one reading per patient per 5 min) → scale horizontally |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Medical data | Patient glucose readings = medical PHI — encrypted at rest (InfluxDB encryption) |
| Model access | Only authorized services can query predictions (internal auth) |
| Audit log | Every prediction stored with model version — full traceability |
| Rollback safety | Never delete previous model version until new one validated clinically |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Inference latency | < 100ms for single patient forecast (CPU ONNX) |
| Batch prediction | Pre-compute 30-min forecast for all active patients every 5 min |
| InfluxDB queries | Index on user_id + time range for fast window retrieval |
| Concurrency | Async FastAPI + connection pool to InfluxDB |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Model Accuracy | RMSE < 20 mg/dL (30-min horizon) on held-out patient data |
| MAPE Tests | < 15% MAPE on test set before model promotion |
| API Tests | Forecast endpoint returns valid JSON with correct horizon values |
| Drift Tests | Inject degraded input → verify drift alert triggered |
| Rollback Tests | Verify previous model version restorable in < 5 minutes |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | MAPE per patient, RMSE distribution, inference latency, prediction request rate |
| Drift alerts | MAPE > 15% → Prometheus alert → Grafana → retrain trigger |
| Logging | Every prediction: user_id, model_version, horizon, predicted_value, timestamp |
| Tracing | OpenTelemetry trace from CGM data ingestion → prediction → risk alert |

---

## Research Complexity

| Level | Justification |
|---|---|
| 🔴 Advanced | Medical-grade ML: clinical accuracy requirements, drift monitoring, model versioning with rollback, LSTM + Transformer ensemble, per-patient personalization |

---

## Portfolio Value

| Skill Demonstrated |
|---|
| Bi-LSTM for medical time-series forecasting |
| Temporal Fusion Transformer for multi-horizon prediction |
| LSTM + Transformer ensemble design |
| Clinical model drift monitoring (MAPE) |
| MLflow model registry with promotion gates |
| InfluxDB time-series database for health data |
| Medically critical service architecture (independent rollback) |
| Federated learning model consumption |
