# ⌚ Repo 18 — `wearable-sync-service`

> **Status:** Planned · **Flag:** — · **Difficulty:** ⭐⭐⭐ · **Build Priority:** 11 (parallel with 17)

---

## Purpose

The **BLE smart band data collection and activity relay server**. Manages the entire lifecycle of connecting BLE wearable devices (smart bands / fitness watches) to GlucoVision — pairing, bonding, reading health metrics (heart rate, step count, SpO2), and relaying real-time activity data to the recommendation and glucose prediction services. Separated from the ESP32-CAM streaming service (`17`) because BLE protocol handling, GATT characteristic parsing, and activity data normalisation are a completely different engineering domain from camera firmware and video streaming.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `ble_gateway/` | BLE central role: scans, connects, and bonds to smart bands via flutter_blue_plus (mobile) or BlueZ (Linux relay) |
| `gatt_parser/` | Parses BLE GATT characteristics: heart rate (0x2A37), step count (custom), SpO2 (0x2A5F), battery (0x2A19) |
| `band_profiles/` | Per-device profile library: Xiaomi Mi Band, Fitbit, Garmin, generic HRS bands |
| `activity_normaliser/` | Normalises step counts, HR values, SpO2 from different band formats into a unified schema |
| `activity_relay/` | REST + MQTT relay — pushes normalised activity data to `15` recommendation engine and `12` glucose prediction |
| `sync_scheduler/` | Triggered sync on patient app open + background BLE poll every 5 minutes |
| `device_registry/` | REST API: register, list, manage paired smart bands per patient |
| `realtime_stream/` | WebSocket stream of live HR data during exercise sessions |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| BLE GATT characteristic notifications | Smart band hardware | BLE binary GATT packets |
| Device pairing request | Patient via mobile app (03) | BLE scan + bond |
| Manual sync trigger | Mobile app (03) | REST request |
| Registered device list | PostgreSQL | JSON |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Activity data (steps, HR, SpO2, calories) | `15` recommendation-engine | JSON REST |
| Activity time-series | `12` glucose-prediction | JSON REST |
| Activity time-series (persistent) | InfluxDB | InfluxDB line protocol |
| Live HR stream (during exercise) | Mobile app (03) | WebSocket JSON |
| Sync status | Mobile app (03) | JSON `{ last_sync, device_name, battery_pct }` |
| Band telemetry | `08` monitoring | JSON / MQTT |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `08` api-gateway | REST routing + auth forwarding |
| `15` recommendation-engine | Consumes activity data for TDEE and meal plan updates |
| `12` glucose-prediction | Activity as conditioning signal for glucose forecast |
| `06` user-service | Patient device registration and preferences |
| InfluxDB | Activity time-series storage |
| MQTT Broker | Band telemetry publishing (optional, for battery alerts) |
| flutter_blue_plus | BLE scanning and GATT operations on Android/iOS |

---

## Communication

| Protocol | Used For |
|---|---|
| Bluetooth LE (GATT) | Smart band → mobile app: health data notifications |
| REST / HTTPS | Activity relay to `15`, `12`; device registry API |
| WebSocket | Live HR stream to mobile during exercise sessions |
| InfluxDB client | Write activity time-series |
| MQTT (optional) | Band battery alert publishing |

---

## Tech Stack

| Layer | Technology |
|---|---|
| BLE (Mobile) | Flutter `flutter_blue_plus` plugin — scanning, GATT, bonding |
| BLE (Linux/Server) | Python `bleak` library — headless BLE relay option |
| GATT Parsing | Python `construct` library for binary GATT packet parsing |
| REST API | FastAPI (Python) — activity relay, device registry, sync status |
| WebSocket | FastAPI WebSocket — live HR during exercise |
| Time-Series DB | InfluxDB |
| Database | PostgreSQL — device registry, sync history |
| Containerization | Docker (relay API + optional BlueZ container) |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| Step counting | Raw accelerometer peak detection runs on-band; service receives already-counted steps |
| HR zones | Service classifies HR into zones: resting / fat-burn / cardio / peak — used by `15` for exercise intensity |
| Activity factor | Step count + HR zone → activity factor multiplier for TDEE in `15` energy engine |
| SpO2 relevance | Low SpO2 (< 94%) combined with glucose alert → severity escalation in `13` |
| Glucose–activity correlation | Activity data timestamped and sent to `12` — high intensity exercise increases insulin sensitivity |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| InfluxDB | Activity time-series (steps/min, HR, SpO2) | `measurement=activity, user_id=tag, source=band_model` |
| PostgreSQL | Device registry (device_id, mac_address, patient_id, band_model, last_sync) | Relational |

**InfluxDB activity schema:**
```
measurement: activity_readings
tags:
  user_id: "patient-uuid"
  source: "mi_band_8 | fitbit_charge6 | generic_hrs"
  device_mac: "AA:BB:CC:DD:EE:FF"
fields:
  steps_per_min: 82       (integer)
  heart_rate_bpm: 74      (integer)
  spo2_pct: 98            (float)
  calories_kcal: 12.4     (float)
  battery_pct: 65         (integer)
timestamp: nanoseconds
```

**Unified activity event (REST to `15`, `12`):**
```json
{
  "user_id": "patient-uuid",
  "timestamp": "2026-05-10T08:30:00Z",
  "steps_total": 4200,
  "avg_heart_rate_bpm": 78,
  "hr_zone": "fat_burn",
  "spo2_pct": 97,
  "calories_kcal": 310,
  "active_minutes": 45,
  "source": "mi_band_8"
}
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| BLE range | BLE 5.0: 10–30m range — mobile app acts as BLE central, relays to server |
| Background sync | Flutter WorkManager: background BLE sync every 5 min when app backgrounded |
| Multiple bands | One patient can have one active band; service enforces single primary device |
| Band compatibility | Profile library supports Xiaomi Mi Band 6/7/8, generic BLE HRS (Heart Rate Service 0x180D) |
| Server-side BLE | Optional: BlueZ on Linux for server-side BLE (useful for testing without mobile) |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| BLE pairing | BLE bonding (encrypted GATT, passkey or Just Works depending on band capability) |
| PHI | HR, steps, SpO2 = health data — encrypted in InfluxDB at rest |
| Auth | JWT validation on all REST endpoints via gateway |
| MAC address | Device MAC stored hashed — not in plaintext logs |
| Activity data scope | Only paired patient's band data accepted — reject unknown MAC addresses |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| BLE polling interval | 5-second interval for GATT notify; 60-second for polling mode bands |
| Battery impact | BLE 5.0 LE: < 5mA active scanning — acceptable for phone battery |
| InfluxDB write | Batch writes every 30 seconds (buffer GATT readings) |
| Live HR WebSocket | Real-time GATT notification → WebSocket forward: < 200ms latency |
| Sync on open | Full data sync on app open (pull last 1 hour of band storage) |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| GATT Parser Tests | Unit tests for each band profile — parse known binary packets → assert correct values |
| BLE Integration Tests | flutter_blue_plus integration test with physical band in lab |
| Activity Relay Tests | Post activity JSON → verify InfluxDB write and `15` receives correct payload |
| HR Zone Tests | HR = 145 bpm → assert zone = "cardio" |
| Sync Tests | Simulate band reconnect after 2-hour gap → verify full gap backfill |
| WebSocket Tests | Live HR stream stable over 30-minute exercise session |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Connected bands count, sync success rate, avg sync latency, BLE disconnect rate |
| Logging | Device connect/disconnect, sync events with patient_id, readings received count |
| Alerts | Band disconnected > 30 min → push notification; battery < 15% → notify patient |
| InfluxDB dashboards | Per-patient daily step trend, resting HR trend (via `08` Grafana) |

---

## Research Complexity

**🟡 Intermediate** — BLE GATT protocol parsing, multi-device compatibility profiles, activity data normalisation for ML input, background sync on mobile.

---

## Portfolio Value

BLE GATT protocol implementation (flutter_blue_plus / bleak) · Multi-device smart band compatibility layer · Activity data normalisation for ML pipeline input · InfluxDB time-series ingestion for health metrics · Background BLE sync scheduling (Flutter WorkManager) · Real-time HR WebSocket stream during exercise · PHI-compliant health wearable data handling
