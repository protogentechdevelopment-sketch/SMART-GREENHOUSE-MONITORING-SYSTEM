# 🌿 SMART-GREENHOUSE-MONITORING-SYSTEM

> An IoT + edge-AI system for automated greenhouse management using ESP32-based climate control and ESP32-CAM-based ML plant disease detection — built on ESP32 + Edge Impulse.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Communication Protocols](#communication-protocols)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Traditional greenhouse systems rely on manual monitoring of temperature, humidity, and soil moisture — causing delayed responses, resource wastage, and missed disease outbreaks. This project eliminates those limitations with a **fully automated, IoT-driven greenhouse system** built on embedded hardware.

When environmental conditions deviate from thresholds, the ESP32 instantly activates actuators to restore optimal conditions. Simultaneously, an ESP32-CAM mounted on a motorized rail scans plant leaves and runs an **on-device Edge Impulse ML model** for real-time disease classification — **no cloud, no manual inspection, no cashier.**

---

## Features

- ✅ Real-time temperature and humidity monitoring via DHT11 sensor
- ✅ Soil moisture sensing using Capacitive Soil Moisture Sensor v2.0
- ✅ Automated actuator control — humidifier + water pump via 5V relay module
- ✅ On-device plant disease detection using Edge Impulse ML model on ESP32-CAM
- ✅ Motorized 180° camera scanning — 10 RPM DC gear motor + SG90 servo
- ✅ Real-time status on 16×2 LCD display (temperature, humidity, disease result)
- ✅ UART communication between ESP32-CAM and main ESP32 MCU
- ✅ Edge-based inference — no internet required for disease classification
- ✅ Wi-Fi-capable for future cloud/IoT integration via ESP32

---

## System Architecture

```
┌───────────────────────────────────────────────────────────┐
│  INPUT LAYER       DHT11 → Temperature + Humidity         │
│                    Capacitive Sensor → Soil Moisture      │
│                    ESP32-CAM → Plant Leaf Images          │
├───────────────────────────────────────────────────────────┤
│  PROCESSING LAYER  ESP32 (Main MCU — Climate Control)     │
│                    Arduino framework (C/C++, loop/setup)  │
├───────────────────────────────────────────────────────────┤
│  INFERENCE LAYER   ESP32-CAM + Edge Impulse ML Model      │
│                    UART → Disease label → Main ESP32      │
├───────────────────────────────────────────────────────────┤
│  OUTPUT LAYER      16×2 LCD Display                       │
│                    Humidifier + Water Pump (via relay)    │
│                    Motorized camera (gear motor + servo)  │
└───────────────────────────────────────────────────────────┘
```

### Block Diagram

```
DHT11 ──────────────────────────────────────┐
                                             ▼
Capacitive Moisture Sensor ──────► ESP32 (Main MCU) ──► 5V Relay ──► Humidifier
                                        │                         └──► Water Pump
                                        ├──► 16×2 LCD Display
                                        │
                                        └──► UART ◄──── ESP32-CAM
                                                           │
                                                    OV2640 Camera
                                                           │
                                               Edge Impulse ML Model
                                                           │
                                              L293D ──► DC Gear Motor
                                              SG90 Servo Motor
```

---

## Hardware Components

| Component | Model | Interface | Role |
|---|---|---|---|
| Main MCU | ESP32 (Xtensa LX6, 240 MHz, Wi-Fi/BT) | — | Climate control, sensor fusion, UART |
| Camera MCU | ESP32-CAM (OV2640 sensor) | UART | Image capture + on-device ML inference |
| Temperature & Humidity | DHT11 | Single-wire digital | Measures temp (0–50°C ±2°C) + humidity |
| Soil Moisture | Capacitive Soil Moisture Sensor v2.0 | ADC (analog) | Measures soil moisture level |
| Relay Module | 5V Single-channel Relay | GPIO | Switches humidifier and water pump |
| Humidifier | 5VDC 108kHz Ultrasonic Mist Maker | GPIO via relay | Raises air humidity via ultrasonic atomization |
| Water Pump | DC Mini Submersible Pump (5V/12V) | GPIO via relay | Irrigates soil when moisture drops |
| LCD Display | 16×2 LCD (HD44780, 4-bit parallel) | GPIO | Shows temp, humidity, actuator and disease status |
| Gear Motor | 10 RPM DC Gear Motor | L293D driver | Moves ESP32-CAM linearly along plant rail |
| Servo Motor | SG90 (0°–180°, PWM) | PWM GPIO | Adjusts camera vertical angle |
| Motor Driver | L293D H-bridge | GPIO | Controls DC gear motor direction and speed |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **Arduino IDE** | ESP32 + ESP32-CAM firmware development in C/C++ |
| **Edge Impulse** | ML model training, optimization, and Arduino library export |
| **Kaggle** | Source of labeled plant disease image datasets (healthy + diseased) |
| **imgBB API** | Optional cloud upload of captured disease images via HTTPS |

---

## Communication Protocols

### Single-Wire Serial — ESP32 ↔ DHT11

| ESP32 Pin | DHT11 Pin | Signal |
|---|---|---|
| GPIO (data pin) | OUT | Calibrated 40-bit temp + humidity stream |
| 3.3V | VCC | Power |
| GND | GND | Ground |

### ADC — ESP32 ↔ Capacitive Soil Moisture Sensor

| ESP32 Pin | Sensor Pin | Signal |
|---|---|---|
| GPIO34 / GPIO35 | OUT | Analog voltage proportional to moisture |
| 3.3V / 5V | VCC | Power |
| GND | GND | Ground |

### GPIO — ESP32 ↔ 5V Relay Module

- ESP32 sends digital HIGH/LOW to relay IN pin
- Relay switches humidifier and water pump loads (electrically isolated)
- Relay LED indicator shows ON/OFF state

### UART — ESP32-CAM ↔ Main ESP32

- Baud: **9600**, 8N1
- TX pin: GPIO14, RX pin: GPIO15 (ESP32-CAM)
- ESP32-CAM transmits: disease label + confidence (e.g., `rust 87.3%`)
- Main ESP32 receives and displays result on LCD

### PWM — ESP32 ↔ SG90 Servo

- 50 Hz PWM signal via GPIO
- Pulse width 500 µs = 0°, 1500 µs = 90°, 2400 µs = 180°
- ESP32Servo library used for non-blocking servo sweep

---

## Circuit & Pin Connections

### DHT11 Pin Configuration

| Pin | Function | Description |
|---|---|---|
| VCC | Power | Supply voltage (3.3V/5V) |
| GND | Ground | Common ground |
| OUT | Signal | Single-wire digital output to ESP32 |

### Capacitive Soil Moisture Sensor Pin Configuration

| Pin | Function | Description |
|---|---|---|
| VCC | Power | 3.3V/5V supply |
| GND | Ground | Common ground |
| OUT | Signal | Analog/Digital output to ESP32 ADC pin |

### Relay Pin Configuration

| Pin | Function | Description |
|---|---|---|
| VCC | Power | 5V supply |
| GND | Ground | Common ground |
| IN | Control | Input signal from ESP32 |
| COM | Common | Common terminal |
| NO | Normally Open | Connected when relay is active |
| NC | Normally Closed | Connected when relay is inactive |

### SG90 Servo Pin Configuration

| Wire | Function | Description |
|---|---|---|
| Red | Power | 5V supply |
| Brown | Ground | Common ground |
| Orange | Signal | PWM input from ESP32 |

### Humidifier Pin Configuration

| Pin | Function | Description |
|---|---|---|
| VCC (5V) | Power Supply (+) | Connect to regulated 5V DC supply |
| GND | Ground | Common ground with ESP32 and relay |
| EN / CTRL | Control Input | Used to turn ON/OFF module via relay |

### Weight Calculation (Soil Moisture Mapping)

```
ADC Raw → Mapped to moisture % using calibrated threshold:
  Dry   = Raw > upper_threshold
  Moist = lower_threshold < Raw < upper_threshold
  Wet   = Raw < lower_threshold

Example:
  Raw ADC = 2800 → Dry → Water pump activated
  Raw ADC = 1500 → Moist → Pump off
```

---

## How It Works

1. **Boot** — ESP32 initializes DHT11, capacitive moisture sensor, relay GPIOs, LCD, UART. ESP32-CAM initializes OV2640 camera, Wi-Fi (optional), UART2, and Edge Impulse inferencing.
2. **Environmental Monitoring** — DHT11 reads temperature + humidity every loop cycle. Capacitive sensor reads soil moisture via ADC.
3. **Climate Control** — ESP32 compares readings against predefined thresholds. Low humidity → relay activates humidifier. Low soil moisture → relay activates water pump. Actuators turn off once target conditions are restored.
4. **Camera Scanning** — Gear motor moves ESP32-CAM forward/backward along plant rail (limit switches at both ends). SG90 servo sweeps 0°–180° while motor moves, giving comprehensive leaf coverage.
5. **ML Inference** — ESP32-CAM captures JPEG frames, runs Edge Impulse image classification model on-device. Model outputs label (e.g., `healthy`, `rust`, `diseased`) with confidence score.
6. **Data Transfer** — ESP32-CAM transmits disease label + confidence over UART at 9600 baud to main ESP32. Optional: JPEG uploaded to imgBB via HTTPS if Wi-Fi is connected.
7. **Display Update** — Main ESP32 renders temperature, humidity, actuator status, and disease detection result on 16×2 LCD in real time.

### ESP32-CAM ML Inference Flow

```
Camera initialized (OV2640, QVGA 320×240)
  └─► Frame captured as JPEG
        └─► Converted to RGB888 for Edge Impulse
              └─► run_classifier(&signal, &result)
                    └─► Best label + confidence extracted
                          └─► Confidence ≥ 0.7 → UART transmit + imgBB upload
                          └─► Confidence < 0.7 → SKIPPED (low confidence)
```

### ESP32 Firmware Snippet (Actuator Control)

```cpp
void loop() {
    float temp = dht.readTemperature();
    float hum  = dht.readHumidity();
    int moisture = analogRead(MOISTURE_PIN);

    if (hum < HUMIDITY_THRESHOLD) {
        digitalWrite(HUMIDIFIER_RELAY, LOW);  // activate
    } else {
        digitalWrite(HUMIDIFIER_RELAY, HIGH); // deactivate
    }

    if (moisture > DRY_THRESHOLD) {
        digitalWrite(PUMP_RELAY, LOW);        // activate
    } else {
        digitalWrite(PUMP_RELAY, HIGH);       // deactivate
    }

    lcd.setCursor(0, 0);
    lcd.print("T:" + String(temp) + "C H:" + String(hum) + "%");
}
```

### ESP32-CAM UART Transmit Snippet

```cpp
void sendDiseaseUART(const char* label, float confidence) {
    String msg = String(label) + " " + String(confidence * 100.0f, 1) + "%";
    Serial2.println(msg);
}
```

---

## Results

| Metric | Traditional (Manual) | Proposed System |
|---|---|---|
| Climate monitoring | Periodic manual checks | Continuous real-time |
| Actuator response | Manual + delayed | Automatic, threshold-based |
| Disease detection | Visual inspection (subjective) | ML model, on-device inference |
| Disease detection latency | Hours–days | Seconds per scan cycle |
| Camera coverage | Single viewpoint | 180° multi-angle motorized scan |
| Internet dependency | — | None (fully offline inference) |
| Resource wastage | High (over-irrigation common) | Minimized (sensor-triggered only) |

- Dual-parameter verification (temperature + humidity + moisture) ensures precise actuator control with no over-irrigation or over-humidification.
- Edge-based ML eliminates cloud latency — classification runs directly on ESP32-CAM.
- Motorized scanning with gear motor + servo provides comprehensive leaf coverage that static camera setups cannot achieve.

---

## Future Scope

- **Cloud / IoT** — Push sensor data and disease events to AWS IoT / ThingSpeak / Firebase via ESP32 Wi-Fi for historical analytics
- **Mobile App** — Real-time dashboard on farmer's smartphone with push alerts for disease detection and actuator status
- **Expanded Sensor Suite** — Add CO₂ sensor, light intensity (LDR/BH1750), and soil NPK sensor for full precision agriculture coverage
- **Advanced ML** — Retrain with larger Kaggle datasets + deep learning (MobileNetV2) for multi-disease classification with higher accuracy
- **Solar Power** — Integrate solar panel + battery backup for off-grid greenhouse deployment
- **Stepper Motor** — Replace DC gear motor with stepper for precise, position-controlled camera movement
- **Security** — Implement TLS/SSL encryption on UART + Wi-Fi payloads; add authentication for cloud endpoints

> *Merging embedded intelligence with precision agriculture — automated climate, zero manual disease detection.*
