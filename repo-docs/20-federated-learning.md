# 🌐 Repo 20 — `federated-learning`

> **Status:** Planned · **Flag:** 🔬 Privacy Critical · **Difficulty:** ⭐⭐⭐⭐⭐ · **Build Priority:** 19

---

## Purpose

The **privacy-preserving collaborative model training system** — the feature that allows GlucoVision models to improve from all patients' data without any patient's data ever leaving their device or institution. Using Flower (flwr) with differential privacy, multiple hospitals or research institutions can train a shared glucose prediction model without sharing raw patient records.

**Regulatory basis:** GDPR Article 89 (research exemption with safeguards) + HIPAA Safe Harbor — raw patient data never transmitted.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `fl_server/` | Flower FL server — coordinates training rounds, aggregates updates via FedAvg / FedProx |
| `fl_client/` | Per-institution training client SDK — trains local model, sends gradient update only |
| `privacy/` | Differential privacy (ε-DP) — adds calibrated Gaussian noise to gradients before transmission |
| `secure_agg/` | Secure aggregation — server aggregates encrypted gradients (cannot inspect individual updates) |
| `experiment_tracker/` | FL round metrics: convergence, per-client loss, privacy budget consumption |
| `model_distributor/` | Pushes aggregated model to MLflow registry for `09` and `12` to consume |
| `client_sdk/` | Python package (`glucovision-fl-client`) installable by participating institutions |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Local model gradient updates | FL clients (hospitals / patient devices) | Encrypted gradient tensors |
| Global model weights (round start) | Flower FL server → clients | PyTorch state_dict |
| Training participation confirmation | Participating institutions | REST API enrollment |
| Privacy budget configuration | Platform admin | JSON `{ epsilon, delta, noise_multiplier }` |
| FL round configuration | Admin via `04` web dashboard | JSON `{ num_rounds, min_clients, strategy }` |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Aggregated global model | MLflow registry → `09` food-recognition, `12` glucose-prediction | PyTorch state_dict |
| FL round metrics | `04` web dashboard | JSON `{ round, loss, accuracy, clients_participated }` |
| Privacy budget report | Compliance / admin | JSON `{ epsilon_spent, delta, rounds }` |
| Client SDK | Participating institutions | Python package (pip install) |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `09` food-recognition | Source model for food classification FL training |
| `12` glucose-prediction | Source model for glucose forecasting FL training |
| MLflow | Aggregated model registry |
| Flower (flwr) | FL framework — server + client coordination |
| PyTorch | Model training and gradient computation |
| Opacus | PyTorch differential privacy library |
| OpenMined PySyft | Secure aggregation protocol |
| `08` api-gateway | Client connections to FL server |
| `04` web dashboard | FL round monitoring UI |

---

## Communication

| Protocol | Used For |
|---|---|
| gRPC (Flower default) | FL server ↔ FL clients — model weights + gradient exchange |
| HTTPS / REST | Enrollment API, metrics API |
| MLflow API | Push aggregated model to registry |
| Mutual TLS | All FL communication encrypted end-to-end |

---

## Tech Stack

| Layer | Technology |
|---|---|
| FL Framework | Flower (flwr) 1.x |
| Deep Learning | PyTorch 2.x |
| Differential Privacy | Opacus (PyTorch DP library) |
| Secure Aggregation | OpenMined PySyft / custom SecAgg |
| Experiment Tracking | MLflow |
| Communication | gRPC (built into Flower) |
| Backend API | FastAPI (Python) — enrollment + metrics |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **FedAvg** | Weighted average of client model weights by dataset size |
| **FedProx** | Adds proximal term to handle client heterogeneity (different patient populations) |
| **Differential Privacy** | Opacus clips per-sample gradients, adds Gaussian noise. Guarantee: (ε=1.0, δ=1e-5)-DP per round |
| **Privacy budget** | Rényi DP accountant tracks total ε spent across rounds — alert when approaching limit |
| **Secure Aggregation** | Server sees only the sum of encrypted gradients — cannot inspect individual updates |
| **Non-IID data** | Patient data is non-identically distributed (ethnicity, lifestyle) — FedProx handles this |
| **FL for food recognition** | Each institution fine-tunes `09` ResNet-50 on local food dataset → global model improves |
| **FL for glucose prediction** | Each institution trains `12` LSTM on local CGM data → aggregated without sharing PHI |

**DP noise equation:**
```
gradient_noised = clip(gradient, C) + N(0, σ²C²)
where:
  C  = gradient clipping threshold
  σ  = noise multiplier (higher = more private, less accurate)
  ε  ∝ T / σ²   (T = number of rounds)
```

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| MLflow | Global model versions, FL round metrics | Binary (PyTorch) + JSON |
| PostgreSQL | Enrolled clients, round participation, privacy budget ledger | Relational |
| Local client only | Raw patient training data — never leaves institution | Local filesystem |

**Privacy budget ledger:**
```json
{
  "client_id": "hospital-uuid",
  "model": "glucose_prediction_v2",
  "total_rounds": 50,
  "epsilon_spent": 0.85,
  "delta": 1e-5,
  "noise_multiplier": 1.1,
  "gradient_clip_C": 1.0,
  "budget_limit": 2.0,
  "status": "active"
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| FL server | Persistent Flower server; clients connect per training round |
| Client deployment | Docker container at each participating institution |
| gRPC port | FL server on port 8080 — firewall rules required |
| Round scheduling | Nightly FL rounds (avoid peak hours) |
| Client SDK | `pip install glucovision-fl-client` → institution integrates in training pipeline |
| Regulatory isolation | Separate environment — cannot share infrastructure with production services |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| No raw data | Gradient updates only — raw patient PHI never leaves institution |
| Mutual TLS | FL server + client mutual TLS authentication |
| Gradient encryption | SecAgg: gradients encrypted before sending; server decrypts aggregate only |
| Differential privacy | Mathematical guarantee: individual patient contribution indistinguishable in aggregate |
| Audit trail | Every FL round logged: clients participated, rounds completed, ε budget spent |
| GDPR / HIPAA | DP provides regulatory-grade privacy proof; data minimisation enforced |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| Communication overhead | Gradient size ≈ model size (ResNet-50: 25MB) → FP16 quantize before sending (50% reduction) |
| Round duration | 50 clients × 1 local epoch each → ~10 min per round on GPU |
| Aggregation latency | FedAvg aggregation < 1 sec on server |
| Stragglers | Flower waits for min_clients threshold (e.g., 80%) before aggregating |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Simulation Tests | Flower simulation mode — verify FedAvg convergence in 10 rounds |
| DP Tests | Verify Opacus noise doesn't corrupt model (accuracy within 2% of non-private baseline) |
| SecAgg Tests | Verify server cannot reconstruct individual gradients |
| Client SDK Tests | `pip install glucovision-fl-client` → training → gradient submission roundtrip |
| Budget Tests | Run 100 rounds → verify ε budget tracked correctly |
| MLflow Tests | Aggregated model pushed and accessible by `09`, `12` |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Rounds completed, clients per round, global loss, ε budget consumed |
| FL Dashboard | `04` web dashboard shows convergence curve, participating institutions |
| Alerts | < 50% client participation → round failed; ε > 80% → reduce rounds |
| MLflow | Full FL experiment run history (round → metrics → model artifact) |

---

## Research Complexity

**🔴 Expert** — Active ML research area; differential privacy requires mathematical understanding of ε-δ guarantees; secure aggregation is cryptographic; non-IID data handling is an open research problem.

---

## Portfolio Value

Federated learning system (Flower) · Differential privacy with Opacus (ε-DP) · FedAvg and FedProx aggregation · Secure gradient aggregation (SecAgg) · Privacy budget tracking (Rényi DP accountant) · GDPR/HIPAA-compliant ML pipeline · Distributable FL client SDK · Non-IID federated learning for patient population heterogeneity · MLflow model registry integration
