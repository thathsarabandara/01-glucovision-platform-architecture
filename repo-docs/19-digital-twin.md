# 🧬 Repo 19 — `digital-twin`

> **Status:** Planned · **Flag:** 🔬 Research Grade · **Difficulty:** ⭐⭐⭐⭐⭐ · **Build Priority:** 18

---

## Purpose

The **patient's physiological digital twin** — a living computational model of a specific patient's metabolic system. Using Neural Ordinary Differential Equations (Neural ODE), this service simulates how a patient's glucose, HbA1c, BMI, and energy metabolism evolve over time, enabling **what-if scenario exploration**: *"What happens to my glucose if I eat hoppers for breakfast instead of string hoppers tomorrow?"*

Separated from production inference services because Neural ODE simulation is experimental with its own training lifecycle and different compute profile from static model inference.

**Research basis:** Neural ODE physiological modeling is identified as a frontier research direction in the systematic review for adaptive patient simulation.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `twin_engine/` | Neural ODE patient state model — models glucose dynamics, BMI trend, HbA1c evolution |
| `simulation_api/` | What-if scenario runner: given a proposed meal or exercise, simulate future glucose curve |
| `forecasting/` | 24–72 hour health projection (glucose, energy balance, weight trend) |
| `data_sync/` | Continuously updated from `12` glucose prediction and `15` recommendation data |
| `viz_api/` | Chart data endpoints for mobile (03) and web dashboard (04) |
| `scenario_store/` | Save and compare multiple what-if scenarios per patient |
| `calibration/` | Per-patient Neural ODE parameter fitting using personal glucose history (adjoint sensitivity) |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| Glucose prediction data | `12` glucose-prediction | JSON time-series / InfluxDB |
| Recommendation context | `15` recommendation-engine | JSON (meal plan, TDEE) |
| Historical glucose readings | InfluxDB (via `14`) | Time-series |
| Patient health metadata | `06` user-service | JSON (BMI, medications, diabetes type) |
| What-if scenario request | Mobile app (03), Web dashboard (04) | JSON `{ meal: {...}, exercise: {...}, horizon_hours: 24 }` |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| 24–72 hr glucose projection | Mobile app (03) | JSON time-series (chart data) |
| HbA1c trend forecast | Web dashboard (04) | JSON |
| BMI evolution curve | Web dashboard (04) | JSON |
| What-if scenario result | Mobile app (03), Web dashboard (04) | JSON `{ baseline_curve, scenario_curve, delta }` |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `12` glucose-prediction | Real-time glucose data feed for twin calibration |
| `15` recommendation-engine | Meal and energy context for simulation |
| `06` user-service | Patient physiological parameters |
| InfluxDB | Historical glucose time-series for ODE parameter fitting |
| Redis | Cache current twin state per patient (fast scenario evaluation) |
| `08` api-gateway | REST routing |

---

## Communication

| Protocol | Used For |
|---|---|
| REST / HTTPS | Scenario API, forecast API, viz data API |
| Internal HTTP | Receives updates from `12`, `15` |
| InfluxDB client | Historical glucose read for calibration |
| Redis client | Twin state caching |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | FastAPI (Python) |
| Neural ODE | torchdiffeq (PyTorch ODE solver) |
| Deep Learning | PyTorch 2.x |
| Scientific Computing | SciPy, NumPy |
| Time-Series DB | InfluxDB |
| Cache | Redis |
| Experiment Tracking | MLflow |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| **Neural ODE** | `torchdiffeq.odeint` — learns continuous-time patient physiology dynamics; differential equation parameters are a neural network |
| **Patient state vector** | `[glucose_mmol, insulin_units, carb_absorption_rate, energy_expenditure, bmi, hba1c]` |
| **Parameter fitting** | Per-patient calibration via adjoint sensitivity method — fits ODE parameters to 30-day glucose history |
| **What-if simulation** | Perturb state with proposed meal → integrate ODE forward → output glucose curve |
| **Glucose-insulin model** | Extends Bergman Minimal Model with Neural ODE for non-linear patient-specific dynamics |
| **24–72 hr horizon** | Complementary to `12` (30–120 min): longer strategic planning view |
| **Uncertainty** | Monte Carlo dropout or Gaussian process wrapper for confidence intervals |

**Key equations (conceptual):**
```
dz/dt = f_θ(z, t, u)   # Neural ODE
z(t0) = z0             # Patient initial state
z(T) = odeint(f_θ, z0, t_span, u)

Where:
  z = patient state vector
  u = control inputs (meal, exercise)
  f_θ = neural network with parameters θ
```

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| InfluxDB | Historical glucose for ODE fitting | Time-series |
| Redis | Current patient twin state (fast scenario evaluation) | JSON with TTL |
| PostgreSQL | Saved scenarios, projection results | Relational |
| MLflow | ODE model versions, calibration runs | Binary + JSON |

**Patient twin state (Redis):**
```json
{
  "user_id": "patient-uuid",
  "state": {
    "glucose_mmol": 5.8,
    "insulin_sensitivity": 0.72,
    "carb_absorption_rate": 0.15,
    "energy_expenditure_kcal_hr": 85.0,
    "bmi": 24.5,
    "hba1c_pct": 6.8
  },
  "model_version": "ode-v1.2.1",
  "calibrated_at": "2026-05-10T06:00:00Z",
  "valid_until": "2026-05-11T06:00:00Z"
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| Compute | ODE integration is CPU-intensive; GPU speeds up parameter calibration |
| Twin update frequency | Re-calibrate per patient daily (batch) or on significant glucose event |
| Redis TTL | Twin state valid 24 hours; recompute on expiry |
| Research lifecycle | Model versions evolve independently — MLflow tracks experiment lineage |
| Isolation | Separate deployment from production services — experimental code |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| Patient model | Digital twin contains patient physiology — PHI; encrypted at rest |
| Scenario privacy | What-if scenarios access-controlled per patient |
| Research boundary | Twin parameters must not be shared across patients (each twin is personal) |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| ODE integration | 24hr simulation in ~200ms (SciPy `solve_ivp` / `torchdiffeq.odeint`) |
| Caching | Redis twin state → scenario evaluation in < 100ms |
| Calibration | Daily batch calibration per patient — not on request path |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| ODE Accuracy | Simulate known glucose history → compare to actual (RMSE target < 1 mmol/L over 24hr) |
| What-if Tests | Meal input → output glucose peaks within physiologically plausible range |
| Calibration Tests | ODE parameters converge for test patient within 100 iterations |
| Cache Tests | Redis twin state used on repeated scenario calls |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | ODE calibration RMSE per patient, scenario evaluation latency, twin state freshness |
| Logging | Calibration events, scenario runs with patient_id + model_version |
| Alerts | RMSE > 2 mmol/L → flag for model refit |
| MLflow | Full experiment tracking for every calibration run |

---

## Research Complexity

**🔴 Expert** — Neural ODE is a research-frontier technique; physiological modeling requires glucose-insulin dynamics domain knowledge; per-patient parameter fitting via adjoint sensitivity method is graduate-level ML research.

---

## Portfolio Value

Neural ODE implementation (torchdiffeq) · Physiological patient state modeling · Bergman Minimal Model extension with neural networks · Per-patient ODE calibration (adjoint sensitivity) · What-if scenario API design · Research-grade ML service architecture · InfluxDB + Redis hybrid for real-time simulation
