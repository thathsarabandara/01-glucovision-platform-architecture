# 📷 Repo 17 — `esp32-cam-streaming-service`

> **Status:** Planned · **Flag:** — · **Difficulty:** ⭐⭐⭐ · **Build Priority:** 11

---

## Purpose

The **ESP32-CAM hardware firmware and video streaming server**. Manages the smart glasses camera hardware lifecycle — WiFi provisioning, OTA updates, image capture — and streams live video via MJPEG/WebRTC to the mobile app and cloud AI services (`09` food recognition, `11` cooking AI). Separated from wearable sync (`18`) because embedded camera firmware and video streaming pipelines are a completely different engineering domain from BLE protocol handling.

---

## Core Functionality

| Module | What It Does |
|---|---|
| `esp32_firmware/` | C++ firmware: WiFi provisioning, BLE config beacon, image capture trigger, deep sleep management |
| `mjpeg_server/` | Lightweight MJPEG HTTP stream server running on ESP32-CAM hardware |
| `webrtc_relay/` | Python aiortc relay — receives MJPEG from ESP32, re-streams via WebRTC to cloud |
| `edge_inference/` | TFLite MobileNetV3 on ESP32-S3 — offline food classification fallback |
| `ota_manager/` | OTA firmware update delivery over HTTPS; binary served from MinIO |
| `mqtt_bridge/` | ESP32 publishes device status + capture events; receives capture commands |
| `device_registry/` | REST API: register, list, manage ESP32-CAM devices per patient |
| `stream_router/` | Routes WebRTC stream to `09` (food log session) or `11` (cooking session) |

---

## Inputs

| Input | Source | Format |
|---|---|---|
| MJPEG video frames | ESP32-CAM hardware (WiFi) | MJPEG over TCP |
| Capture command | Mobile app (03) or gateway (08) | MQTT message |
| Device provisioning request | Mobile app (03) via BLE | JSON |
| OTA firmware binary | `21` devops-infra CI pipeline | Binary (HTTPS) |
| Stream session type | Mobile app (03) | JSON `{ session: "food_log" \| "cooking" }` |

---

## Outputs

| Output | Consumer | Format |
|---|---|---|
| Live WebRTC video stream | Mobile app (03) for AR preview | WebRTC track |
| Food capture frames | `09` food-recognition-service | JPEG frame via HTTP |
| Cooking video stream | `11` cooking-ai-service | MJPEG / WebRTC |
| On-device food recognition result | Mobile app (03) | JSON (TFLite output) |
| Device telemetry (battery, WiFi RSSI) | `08` monitoring | MQTT → JSON |

---

## Dependencies

| Service / Tool | Role |
|---|---|
| `08` api-gateway | REST routing, WebRTC signalling relay |
| `09` food-recognition | Receives frames for cloud food classification |
| `11` cooking-ai | Receives cooking video stream |
| `06` user-service | Patient device registration |
| MQTT Broker (Mosquitto / EMQX) | Telemetry + command channel |
| coturn TURN server | NAT traversal for home WiFi |
| MinIO | TFLite model storage + OTA firmware binaries |

---

## Communication

| Protocol | Used For |
|---|---|
| MJPEG over TCP | ESP32-CAM → relay server (raw video) |
| WebRTC (via coturn) | Relay → mobile app + cloud (low-latency video) |
| MQTT (TLS) | ESP32 telemetry → cloud; commands → ESP32 |
| REST / HTTPS | Device registry API, stream URL endpoint |
| HTTPS | OTA firmware binary delivery |
| BLE NimBLE | Initial WiFi provisioning via mobile app |

---

## Tech Stack

| Layer | Technology |
|---|---|
| ESP32 Firmware | C++ (ESP-IDF) — camera driver, WiFi, NimBLE, OTA |
| Edge ML | TensorFlow Lite — MobileNetV3 int8 quantized (< 500KB) |
| MJPEG Relay | Python + OpenCV |
| WebRTC Server | Python aiortc + aiohttp |
| TURN Server | coturn |
| MQTT Broker | Mosquitto / EMQX |
| REST API | FastAPI (Python) — device registry |
| Containerization | Docker |

---

## AI/ML Knowledge

| Concern | Detail |
|---|---|
| TFLite on ESP32-S3 | MobileNetV3-Small int8 quantized — fits in PSRAM, ~200ms inference |
| Model OTA update | New TFLite model delivered via OTA when `09` produces improved quantized export |
| Cloud fallback | Offline → use on-device result; online → `09` cloud result overrides |
| Frame sampling | For cooking stream: relay samples every 3rd frame before forwarding to `11` |

---

## Data Knowledge

| Storage | Purpose | Format |
|---|---|---|
| ESP32 NVS flash | WiFi credentials, device_id, TFLite model | Binary (NVS) |
| PostgreSQL | Device registry (device_id, patient_id, firmware_version) | Relational |
| MinIO | TFLite model files, OTA firmware binaries | Binary |

**MQTT topic schema:**
```
glucovision/cam/{device_id}/telemetry → { wifi_rssi, battery_pct, fps, uptime_s }
glucovision/cam/{device_id}/event     → { type: "capture", timestamp, food_id_local }
glucovision/cam/{device_id}/command   → { action: "capture" | "stream_start" | "sleep" }
glucovision/cam/{device_id}/ota       → { firmware_url, checksum_sha256 }
```

---

## Infrastructure Knowledge

| Concern | Detail |
|---|---|
| ESP32-S3 | Recommended — PSRAM for TFLite, better WiFi throughput |
| OTA partitions | ESP-IDF dual OTA partition — update one while running from the other |
| TURN server | coturn deployed on public IP — required for WebRTC behind home NAT |
| MJPEG port | ESP32 serves on port 80; relay connects on local WiFi segment |

---

## Security Knowledge

| Concern | Detail |
|---|---|
| MQTT auth | TLS 1.3 + per-device certificates provisioned at setup |
| Firmware signing | OTA binaries signed with RSA-3072; ESP32 verifies before applying |
| WiFi credentials | Provisioned via BLE bond (encrypted GATT) — never hardcoded |
| Video privacy | Frames ephemeral — never stored; only extracted food result persisted |
| No face storage | Filter frames containing faces before forwarding to `09` |

---

## Performance Knowledge

| Concern | Detail |
|---|---|
| MJPEG throughput | ESP32-S3: 15–25 FPS at 640×480 over WiFi |
| WebRTC latency | < 150ms glass-to-mobile with STUN/TURN |
| TFLite inference | ~200ms on ESP32-S3 with PSRAM |
| Relay capacity | aiortc handles 10–20 concurrent WebRTC streams per instance |

---

## Testing Knowledge

| Test Type | Approach |
|---|---|
| Firmware Unit Tests | ESP-IDF unity framework on native_sim target |
| OTA Tests | Full cycle: upload → device verifies signature → reboots → confirm new version |
| Stream Tests | MJPEG relay stability over 30-minute continuous session |
| TFLite Tests | On-device vs cloud `09` disagreement < 10% |
| WebRTC Tests | Peer connection established < 3s including TURN negotiation |

---

## Observability

| Concern | Detail |
|---|---|
| Metrics | Connected devices, stream FPS, OTA success rate, MQTT message rate |
| Logging | Device connect/disconnect, OTA events, firmware version |
| Alerts | Device offline > 10 min → push to patient; battery < 15% → notification |

---

## Research Complexity

**🟠 Advanced** — Embedded C++ (ESP-IDF), real-time WebRTC (aiortc), on-device TFLite inference, OTA update system, MQTT IoT architecture.

---

## Portfolio Value

ESP32 embedded C++ firmware (ESP-IDF) · TFLite edge ML quantization and deployment · WebRTC real-time video pipeline (aiortc + coturn) · MJPEG camera streaming from constrained hardware · MQTT IoT command/telemetry architecture · OTA signed firmware update system · BLE WiFi provisioning · Hardware-to-cloud video pipeline design
